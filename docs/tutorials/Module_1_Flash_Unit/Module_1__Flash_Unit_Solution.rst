Module 1: Flash Unit
====================

In this module, we will familiarize ourselves with the IDAES framework
by creating and working with a flowsheet that contains a single flash
tank. The flash tank will be used to perform separation of Benzene and
Toluene. The inlet specifications for this flash tank are:

Inlet Specifications: \* Mole fraction (Benzene) = 0.5 \* Mole fraction
(Toluene) = 0.5 \* Pressure = 101325 Pa \* Temperature = 368 K

We will complete the following tasks: \* Create the model and the IDAES
Flowsheet object \* Import the appropriate property packages \* Create
the flash unit and set the operating conditions \* Initialize the model
and simulate the system \* Demonstrate analyses on this model through
some examples and exercises

Key links to documentation
--------------------------

-  Main IDAES online documentation page:
   https://idaes-pse.readthedocs.io/en/latest/index.html
-  Core IDAES Library:
   https://idaes-pse.readthedocs.io/en/latest/core/index.html

   -  Flowsheet:
      https://idaes-pse.readthedocs.io/en/latest/core/flowsheet_model.html
   -  Property Packages:
      https://idaes-pse.readthedocs.io/en/latest/core/properties.html
   -  Unit Model:
      https://idaes-pse.readthedocs.io/en/latest/core/unit_model.html

-  Modeling Standards:
   https://idaes-pse.readthedocs.io/en/latest/standards.html

   -  Naming Conventions:
      https://idaes-pse.readthedocs.io/en/latest/standards.html#standard-naming-format

-  IDAES Unit Model Library:
   https://idaes-pse.readthedocs.io/en/latest/models/index.html

Create the Model and the IDAES Flowsheet
----------------------------------------

In the next cell, we will perform the necessary imports to get us
started. From ``pyomo.environ`` (a standard import for the Pyomo
package), we are importing ``ConcreteModel`` (to create the Pyomo model
that will contain the IDAES flowsheet) and ``SolverFactory`` (to create
the object we will use to solve the equations). We will also import
``Constraint`` as we will be adding a constraint to the model later in
the module. Lastly, we also import ``value`` from Pyomo. This is a
function that can be used to return the current numerical value for
variables and parameters in the model. These are all part of Pyomo.

We will also import the main ``FlowsheetBlock`` from IDAES. The
flowsheet block will contain our unit model.

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Execute the cell below to perform the imports. Let a
workshop organizer know if you see any errors.

.. raw:: html

   </div>

.. code:: ipython3

    from pyomo.environ import ConcreteModel, SolverFactory, Constraint, value
    from idaes.core import FlowsheetBlock

In the next cell, we will create the ``ConcreteModel`` and the
``FlowsheetBlock``, and attach the flowsheet block to the Pyomo model.

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Execute the cell below to create the objects

.. raw:: html

   </div>

.. code:: ipython3

    m = ConcreteModel()
    m.fs = FlowsheetBlock(default={"dynamic": False})

At this point, we have a single Pyomo model that contains an (almost)
empty flowsheet block.

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Use the pprint method on the model, i.e. m.pprint(), to
see what is currently contained in the model.

.. raw:: html

   </div>

.. code:: ipython3

    # Todo: call pprint on the model
    m.pprint()


.. parsed-literal::

    1 Block Declarations
        fs : Size=1, Index=None, Active=True
            1 Set Declarations
                time : Dim=0, Dimen=1, Size=1, Domain=None, Ordered=Insertion, Bounds=(0.0, 0.0)
                    [0.0]
    
            1 Declarations: time
    
    1 Declarations: fs


Define Properties
-----------------

We need to define the property package for our flowsheet. In this
example, we have created a property package based on ideal VLE that
contains the necessary components.

IDAES supports creation of your own property packages that allow for
specification of the fluid using any set of valid state variables (e.g.,
component molar flows vs overall flow and mole fractions). This
flexibility is designed to support advanced modeling needs that may rely
on specific formulations. As well, the IDAES team has completed some
general property packages (and is currently working on more). To learn
about creating your own property package, please consult the online
documentation at:
https://idaes-pse.readthedocs.io/en/latest/core/properties.html and look
at examples within IDAES

For this workshop, we will import the BTX_ideal_VLE property package and
create a properties block for the flowsheet. This properties block will
be passed to our unit model to define the appropriate state variables
and equations for performing thermodynamic calculations.

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Execute the following two cells to import and create
the properties block.

.. raw:: html

   </div>

.. code:: ipython3

    import BTX_ideal_VLE as ideal_props

.. code:: ipython3

    m.fs.properties = ideal_props.BTXParameterBlock()

Adding Flash Unit
-----------------

Now that we have the flowsheet and the properties defined, we can create
the flash unit and add it to the flowsheet.

**The Unit Model Library within IDAES includes a large set of common
unit operations (see the online documentation for details:
https://idaes-pse.readthedocs.io/en/latest/models/index.html**

IDAES also fully supports the development of customized unit models
(which we will see in a later module).

Some of the IDAES pre-written unit models: \* Mixer / Splitter \* Heater
/ Cooler \* Heat Exchangers (simple and 1D discretized) \* Flash \*
Reactors (kinetic, equilibrium, gibbs, stoichiometric conversion) \*
Pressure changing equipment (compressors, expanders, pumps) \* Feed and
Product (source / sink) components

In this module, we will import the ``Flash`` unit model from
``idaes.unit_models`` and create an instance of the flash unit,
attaching it to the flowsheet. Each IDAES unit model has several
configurable options to customize the model behavior, but also includes
defaults for these options. In this example, we will specify that the
property package to be used with the Flash is the one we created
earlier.

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Execute the following two cells to import the Flash and
create an instance of the unit model, attaching it to the flowsheet
object.

.. raw:: html

   </div>

.. code:: ipython3

    from idaes.unit_models import Flash

.. code:: ipython3

    m.fs.flash = Flash(default={"property_package": m.fs.properties})

At this point, we have created a flowsheet and a properties block. We
have also created a flash unit and added it to the flowsheet. Under the
hood, IDAES has created the required state variables and model
equations. Everything is open. You can see these variables and equations
by calling the Pyomo method ``pprint`` on the model, flowsheet, or flash
tank objects. Note that this output is very exhaustive, and is not
intended to provide any summary information about the model, but rather
a complete picture of all of the variables and equations in the model.

Set Operating Conditions
------------------------

Now that we have created our unit model, we can specify the necessary
operating conditions. It is often very useful to determine the degrees
of freedom before we specify any conditions.

The ``idaes.core.util.model_statistics`` package has a function
``degrees_of_freedom``. To see how to use this function, we can make use
of the Python function ``help(func)``. This function prints the
appropriate documentation string for the function.

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Import the degrees_of_freedom function and print the
help for the function by calling the Python help function.

.. raw:: html

   </div>

.. code:: ipython3

    # Todo: import the degrees_of_freedom function from the idaes.core.util.model_statistics package
    from idaes.core.util.model_statistics import degrees_of_freedom
    
    # Todo: Call the python help on the degrees_of_freedom function
    help(degrees_of_freedom)


.. parsed-literal::

    Help on function degrees_of_freedom in module idaes.core.util.model_statistics:
    
    degrees_of_freedom(block)
        Method to return the degrees of freedom of a model.
        
        Args:
            block : model to be studied
        
        Returns:
            Number of degrees of freedom in block.
    


.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Now print the degrees of freedom for your model. The
result should be 7.

.. raw:: html

   </div>

.. code:: ipython3

    # Todo: print the degrees of freedom for your model
    print("Degrees of Freedom =", degrees_of_freedom(m))


.. parsed-literal::

    Degrees of Freedom = 7


To satisfy our degrees of freedom, we will first specify the inlet
conditions. We can specify these values through the ``inlet`` port of
the flash unit.

**To see the list of naming conventions for variables within the IDAES
framework, consult the online documentation at:
https://idaes-pse.readthedocs.io/en/latest/standards.html#standard-naming-format**

As an example, to fix the molar flow of the inlet to be 1.0, you can use
the following notation:

.. code:: python

   m.fs.flash.inlet.flow_mol.fix(1.0)

To specify variables that are indexed by components, you can use the
following notation:

.. code:: python

   m.fs.flash.inlet.mole_frac_comp[0, "benzene"].fix(0.5)

.. raw:: html

   <div class="alert alert-block alert-warning">

Note: The “0” in the indexing of the component mole fraction is present
because IDAES models support both dynamic and steady state simulation,
and the “0” refers to a timestep. Dynamic modeling is beyond the scope
of this workshop. Since we are performing steady state modeling, there
is only a single timestep in the model.

.. raw:: html

   </div>

In the next cell, we will specify the inlet conditions. To satisfy the
remaining degrees of freedom, we will make two additional specifications
on the flash tank itself. The names of the key variables within the
Flash unit model can also be found in the online documentation:
https://idaes-pse.readthedocs.io/en/latest/models/flash.html#variables.

To specify the value of a variable on the unit itself, use the following
notation.

.. code:: python

   m.fs.flash.heat_duty.fix(0)

For this module, we will use the following specifications: \* inlet
overall molar flow = 1.0 (``flow_mol``) \* inlet temperature = 368 K
(``temperature``) \* inlet pressure = 101325 Pa (``pressure``) \* inlet
mole fraction (benzene) = 0.5 (``mole_frac_comp[0, "benzene"]``) \*
inlet mole fraction (toluene) = 0.5 (``mole_frac_comp[0, "toluene"]``)
\* The heat duty on the flash set to 0 (``heat_duty``) \* The pressure
drop across the flash tank set to 0 (``deltaP``)

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Write the code below to specify the inlet conditions
and unit specifications described above

.. raw:: html

   </div>

.. code:: ipython3

    # Inlet specifications given above
    m.fs.flash.inlet.flow_mol.fix(1)
    m.fs.flash.inlet.temperature.fix(368)
    m.fs.flash.inlet.pressure.fix(101325)
    m.fs.flash.inlet.mole_frac_comp[0, "benzene"].fix(0.5)
    m.fs.flash.inlet.mole_frac_comp[0, "toluene"].fix(0.5)
    
    # Todo: add code for the 2 flash unit specifications given above
    m.fs.flash.heat_duty.fix(0)
    m.fs.flash.deltaP.fix(0)

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Check the degrees of freedom again to ensure that the
system is now square. You should see that the degrees of freedom is now
0.

.. raw:: html

   </div>

.. code:: ipython3

    # Todo: print the degrees of freedom for your model
    print("Degrees of Freedom =", degrees_of_freedom(m))


.. parsed-literal::

    Degrees of Freedom = 0


Initializing the Model
----------------------

IDAES includes pre-written initialization routines for all unit models.
You can call this initialize method on the units. In the next module, we
will demonstrate the use of a sequential modular solve cycle to
initialize flowsheets.

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Call the initialize method on the flash unit to
initialize the model.

.. raw:: html

   </div>

.. code:: ipython3

    # Todo: initialize the flash unit
    m.fs.flash.initialize()

Now that the model has been defined and intialized, we can solve the
model.

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Using the notation described in the previous model,
create an instance of the “ipopt” solver and use it to solve the model.
Set the tee option to True to see the log output.

.. raw:: html

   </div>

.. code:: ipython3

    # Todo: create the ipopt solver
    solver = SolverFactory('ipopt')
    
    # Todo: solve the model
    status = solver.solve(m, tee=True)


.. parsed-literal::

    Ipopt 3.12.13: 
    
    ******************************************************************************
    This program contains Ipopt, a library for large-scale nonlinear optimization.
     Ipopt is released as open source code under the Eclipse Public License (EPL).
             For more information visit http://projects.coin-or.org/Ipopt
    
    This version of Ipopt was compiled from source code available at
        https://github.com/IDAES/Ipopt as part of the Institute for the Design of
        Advanced Energy Systems Process Systems Engineering Framework (IDAES PSE
        Framework) Copyright (c) 2018-2019. See https://github.com/IDAES/idaes-pse.
    
    This version of Ipopt was compiled using HSL, a collection of Fortran codes
        for large-scale scientific computation.  All technical papers, sales and
        publicity material resulting from use of the HSL codes within IPOPT must
        contain the following acknowledgement:
            HSL, a collection of Fortran codes for large-scale scientific
            computation. See http://www.hsl.rl.ac.uk.
    ******************************************************************************
    
    This is Ipopt version 3.12.13, running with linear solver ma27.
    
    Number of nonzeros in equality constraint Jacobian...:      135
    Number of nonzeros in inequality constraint Jacobian.:        0
    Number of nonzeros in Lagrangian Hessian.............:       53
    
    Total number of variables............................:       41
                         variables with only lower bounds:        3
                    variables with lower and upper bounds:       10
                         variables with only upper bounds:        0
    Total number of equality constraints.................:       41
    Total number of inequality constraints...............:        0
            inequality constraints with only lower bounds:        0
       inequality constraints with lower and upper bounds:        0
            inequality constraints with only upper bounds:        0
    
    iter    objective    inf_pr   inf_du lg(mu)  ||d||  lg(rg) alpha_du alpha_pr  ls
       0  0.0000000e+00 9.76e-09 1.00e+00  -1.0 0.00e+00    -  0.00e+00 0.00e+00   0
    
    Number of Iterations....: 0
    
                                       (scaled)                 (unscaled)
    Objective...............:   0.0000000000000000e+00    0.0000000000000000e+00
    Dual infeasibility......:   0.0000000000000000e+00    0.0000000000000000e+00
    Constraint violation....:   2.1191298905884043e-11    9.7643351182341576e-09
    Complementarity.........:   0.0000000000000000e+00    0.0000000000000000e+00
    Overall NLP error.......:   2.1191298905884043e-11    9.7643351182341576e-09
    
    
    Number of objective function evaluations             = 1
    Number of objective gradient evaluations             = 1
    Number of equality constraint evaluations            = 1
    Number of inequality constraint evaluations          = 0
    Number of equality constraint Jacobian evaluations   = 1
    Number of inequality constraint Jacobian evaluations = 0
    Number of Lagrangian Hessian evaluations             = 0
    Total CPU secs in IPOPT (w/o function evaluations)   =      0.000
    Total CPU secs in NLP function evaluations           =      0.000
    
    EXIT: Optimal Solution Found.
    

.. code:: ipython3

    # For testing purposes
    from pyomo.environ import TerminationCondition
    assert status.solver.termination_condition == TerminationCondition.optimal

Viewing the Results
-------------------

Once a model is solved, the values returned by the solver are loaded
into the model object itself. We can access the value of any variable in
the model with the ``value`` function. For example:

.. code:: python

   print('Vap. Outlet Temperature = ', value(m.fs.flash.vap_outlet.temperature[0]))

You can also find more information about a variable or an entire port
using the ``display`` method from Pyomo:

.. code:: python

   m.fs.flash.vap_outlet.temperature.display()
   m.fs.flash.vap_outlet.display()

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Execute the cells below to show the current value of
the flash vapor outlet pressure. This cell also shows use of the display
function to see the values of the variables in the vap_outlet and the
liq_outlet.

.. raw:: html

   </div>

.. code:: ipython3

    # Print the pressure of the flash vapor outlet
    print('Pressure =', value(m.fs.flash.vap_outlet.pressure[0]))
    
    print()
    print('Output from display:')
    # Call display on vap_outlet and liq_outlet of the flash
    m.fs.flash.vap_outlet.display()
    m.fs.flash.liq_outlet.display()


.. parsed-literal::

    Pressure = 101325.0
    
    Output from display:
    vap_outlet : Size=1
        Key  : Name           : Value
        None :       flow_mol : {0.0: 0.3546244301390833}
             : mole_frac_comp : {(0.0, 'benzene'): 0.6429364285519167, (0.0, 'toluene'): 0.35706357144808326}
             :       pressure : {0.0: 101325.0}
             :    temperature : {0.0: 368.0}
    liq_outlet : Size=1
        Key  : Name           : Value
        None :       flow_mol : {0.0: 0.6453755698609167}
             : mole_frac_comp : {(0.0, 'benzene'): 0.42145852448015175, (0.0, 'toluene'): 0.5785414755198481}
             :       pressure : {0.0: 101325.0}
             :    temperature : {0.0: 368.0}


The output from ``display`` is quite exhaustive and not really intended
to provide quick summary information. Because Pyomo is built on Python,
there are opportunities to format the output any way we like. Most IDAES
models have a ``report`` method which provides a summary of the results
for the model.

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Execute the cell below which uses the function above to
print a summary of the key variables in the flash model, including the
inlet, the vapor, and the liquid ports.

.. raw:: html

   </div>

.. code:: ipython3

    m.fs.flash.report()


.. parsed-literal::

    
    ====================================================================================
    Unit : fs.flash                                                            Time: 0.0
    ------------------------------------------------------------------------------------
        Unit Performance
    
        Variables: 
    
        Key             : Value  : Fixed : Bounds
              Heat Duty : 0.0000 :  True : (None, None)
        Pressure Change : 0.0000 :  True : (None, None)
    
    ------------------------------------------------------------------------------------
        Stream Table
                                  Inlet    Vapor Outlet  Liquid Outlet
        flow_mol                   1.0000      0.35462       0.64538  
        mole_frac_comp benzene    0.50000      0.64294       0.42146  
        mole_frac_comp toluene    0.50000      0.35706       0.57854  
        temperature                368.00       368.00        368.00  
        pressure               1.0132e+05   1.0132e+05    1.0132e+05  
    ====================================================================================


Studying Purity as a Function of Heat Duty
------------------------------------------

Since the entire modeling framework is built upon Python, it includes a
complete programming environment for whatever analysis we may want to
perform. In this next exercise, we will make use of what we learned in
this and the previous module to generate a figure showing some output
variables as a function of the heat duty in the flash tank.

First, let’s import the matplotlib package for plotting as we did in the
previous module.

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Execute the cell below to import matplotlib
appropriately.

.. raw:: html

   </div>

.. code:: ipython3

    import matplotlib.pyplot as plt

Exercise specifications: \* Generate a figure showing the flash tank
heat duty (``m.fs.flash.heat_duty[0]``) vs. the vapor flowrate
(``m.fs.flash.vap_outlet.flow_mol[0]``) \* Specify the heat duty from
-17000 to 25000 over 20 steps

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Using what you have learned so far, fill in the missing
code below to generate the figure specified above. (Hint: import numpy
and use the linspace function from the previous module)

.. raw:: html

   </div>

.. code:: ipython3

    # import the solve_successful checking function from workshop tools
    from workshoptools import solve_successful
    
    # Todo: import numpy
    import numpy as np
    
    # create the empty lists to store the results that will be plotted
    Q = []
    V = []
    
    # create the solver
    solver = SolverFactory('ipopt')
    
    # Todo: Write the for loop specification using numpy's linspace
    for duty in np.linspace(-17000, 25000, 20):
        # fix the heat duty
        m.fs.flash.heat_duty.fix(duty)
        
        # append the value of the duty to the Q list
        Q.append(duty)
        
        # print the current simulation
        print("Simulating with Q = ", value(m.fs.flash.heat_duty[0]))
    
        # Solve the model
        status = solver.solve(m)
        
        # append the value for vapor fraction if the solve was successful
        if solve_successful(status):
            V.append(value(m.fs.flash.vap_outlet.flow_mol[0]))
            print('... solve successful.')
        else:
            V.append(0.0)
            print('... solve failed.')
        
    # Create and show the figure
    plt.figure("Vapor Fraction")
    plt.plot(Q, V)
    plt.grid()
    plt.xlabel("Heat Duty [J]")
    plt.ylabel("Vapor Fraction [-]")
    plt.show()


.. parsed-literal::

    Simulating with Q =  -17000.0
    ... solve successful.
    Simulating with Q =  -14789.473684210527
    ... solve successful.
    Simulating with Q =  -12578.947368421053
    ... solve successful.
    Simulating with Q =  -10368.421052631578
    ... solve successful.
    Simulating with Q =  -8157.894736842105
    ... solve successful.
    Simulating with Q =  -5947.368421052632
    ... solve successful.
    Simulating with Q =  -3736.8421052631566
    ... solve successful.
    Simulating with Q =  -1526.3157894736833
    ... solve successful.
    Simulating with Q =  684.21052631579
    ... solve successful.
    Simulating with Q =  2894.7368421052633
    ... solve successful.
    Simulating with Q =  5105.263157894737
    ... solve successful.
    Simulating with Q =  7315.78947368421
    ... solve successful.
    Simulating with Q =  9526.315789473687
    ... solve successful.
    Simulating with Q =  11736.84210526316
    ... solve successful.
    Simulating with Q =  13947.368421052633
    ... solve successful.
    Simulating with Q =  16157.894736842107
    ... solve successful.
    Simulating with Q =  18368.42105263158
    ... solve successful.
    Simulating with Q =  20578.947368421053
    ... solve successful.
    Simulating with Q =  22789.473684210527
    ... solve successful.
    Simulating with Q =  25000.0
    ... solve successful.



.. image:: output_33_1.png


.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Repeate the exercise above, but create a figure showing
the heat duty vs. the mole fraction of Benzene in the vapor outlet.
Remove any unnecessary printing to create cleaner results.

.. raw:: html

   </div>

.. code:: ipython3

    # Todo: generate a figure of heat duty vs. mole fraction of Benzene in the vapor
    Q = []
    V = []
    
    for duty in np.linspace(-17000, 25000, 20):
        # fix the heat duty
        m.fs.flash.heat_duty.fix(duty)
        
        # append the value of the duty to the Q list
        Q.append(duty)
        
        # solve the model
        status = solver.solve(m)
        
        # append the value for vapor fraction if the solve was successful
        if solve_successful(status):
            V.append(value(m.fs.flash.vap_outlet.mole_frac_comp[0, "benzene"]))
        else:
            V.append(0.0)
            print('... solve failed.')
        
    plt.figure("Purity")
    plt.plot(Q, V)
    plt.grid()
    plt.xlabel("Heat Duty [J]")
    plt.ylabel("Vapor Benzene Mole Fraction [-]")
    plt.show()




.. image:: output_35_0.png


Recall that the IDAES framework is an equation-oriented modeling
environment. This means that we can specify “design” problems natively.
That is, there is no need to have our specifications on the inlet alone.
We can put specifications on the outlet as long as we retain a
well-posed, square system of equations.

For example, we can remove the specification on heat duty and instead
specify that we want the mole fraction of Benzene in the vapor outlet to
be equal to 0.6. The mole fraction is not a native variable in the
property block, so we cannot use “fix”. We can, however, add a
constraint to the model.

Note that we have been executing a number of solves on the problem, and
may not be sure of the current state. To help convergence, therefore, we
will first call initialize, then add the new constraint and solve the
problem. Note that the reference for the mole fraction of Benzene in the
vapor outlet is ``m.fs.flash.vap_outlet.mole_frac_comp[0, "benzene"]``.

.. raw:: html

   <div class="alert alert-block alert-info">

Inline Exercise: Fill in the missing code below and add a constraint on
the mole fraction of Benzene (to a value of 0.6) to find the required
heat duty.

.. raw:: html

   </div>

.. code:: ipython3

    # unfix the heat duty
    m.fs.flash.heat_duty.unfix()
    
    # re-initialize the model - this may or may not be required depending on current state
    m.fs.flash.initialize()
    
    # Todo: Add a new constraint (benzene mole fraction to 0.6)
    m.benz_purity_con = Constraint(expr= m.fs.flash.vap_outlet.mole_frac_comp[0, "benzene"] == 0.6)
    
    # solve the problem
    status = solver.solve(m, tee=True)
    
    # print the value of the heat duty
    print('Q =', value(m.fs.flash.heat_duty[0]))


.. parsed-literal::

    Ipopt 3.12.13: 
    
    ******************************************************************************
    This program contains Ipopt, a library for large-scale nonlinear optimization.
     Ipopt is released as open source code under the Eclipse Public License (EPL).
             For more information visit http://projects.coin-or.org/Ipopt
    
    This version of Ipopt was compiled from source code available at
        https://github.com/IDAES/Ipopt as part of the Institute for the Design of
        Advanced Energy Systems Process Systems Engineering Framework (IDAES PSE
        Framework) Copyright (c) 2018-2019. See https://github.com/IDAES/idaes-pse.
    
    This version of Ipopt was compiled using HSL, a collection of Fortran codes
        for large-scale scientific computation.  All technical papers, sales and
        publicity material resulting from use of the HSL codes within IPOPT must
        contain the following acknowledgement:
            HSL, a collection of Fortran codes for large-scale scientific
            computation. See http://www.hsl.rl.ac.uk.
    ******************************************************************************
    
    This is Ipopt version 3.12.13, running with linear solver ma27.
    
    Number of nonzeros in equality constraint Jacobian...:      137
    Number of nonzeros in inequality constraint Jacobian.:        0
    Number of nonzeros in Lagrangian Hessian.............:       53
    
    Total number of variables............................:       42
                         variables with only lower bounds:        3
                    variables with lower and upper bounds:       10
                         variables with only upper bounds:        0
    Total number of equality constraints.................:       42
    Total number of inequality constraints...............:        0
            inequality constraints with only lower bounds:        0
       inequality constraints with lower and upper bounds:        0
            inequality constraints with only upper bounds:        0
    
    iter    objective    inf_pr   inf_du lg(mu)  ||d||  lg(rg) alpha_du alpha_pr  ls
       0  0.0000000e+00 6.88e-03 1.00e+00  -1.0 0.00e+00    -  0.00e+00 0.00e+00   0
       1  0.0000000e+00 5.39e-02 1.03e-02  -1.0 1.02e+03    -  9.90e-01 1.00e+00H  1
       2  0.0000000e+00 7.45e-09 2.30e-04  -1.0 2.01e-01    -  9.90e-01 1.00e+00h  1
    
    Number of Iterations....: 2
    
                                       (scaled)                 (unscaled)
    Objective...............:   0.0000000000000000e+00    0.0000000000000000e+00
    Dual infeasibility......:   0.0000000000000000e+00    0.0000000000000000e+00
    Constraint violation....:   4.8209533256103537e-12    7.4505805969238281e-09
    Complementarity.........:   0.0000000000000000e+00    0.0000000000000000e+00
    Overall NLP error.......:   4.8209533256103537e-12    7.4505805969238281e-09
    
    
    Number of objective function evaluations             = 4
    Number of objective gradient evaluations             = 3
    Number of equality constraint evaluations            = 4
    Number of inequality constraint evaluations          = 0
    Number of equality constraint Jacobian evaluations   = 3
    Number of inequality constraint Jacobian evaluations = 0
    Number of Lagrangian Hessian evaluations             = 2
    Total CPU secs in IPOPT (w/o function evaluations)   =      0.001
    Total CPU secs in NLP function evaluations           =      0.000
    
    EXIT: Optimal Solution Found.
    Q = 6455.280946055413


.. code:: ipython3

    # For testing purposes
    from pyomo.environ import TerminationCondition
    assert status.solver.termination_condition == TerminationCondition.optimal
