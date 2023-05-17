**[Structure of the Program]{.underline}**

The program uses the *Agents* package
([*https://juliadynamics.github.io/Agents.jl/stable/*](https://juliadynamics.github.io/Agents.jl/stable/))
to simulate collections of cells. Agents provides a generic interface to
build ABM (Agent Based Modelling) applications. The main Agents routine,
**run!**(), calls two customised routines (**update_agent!**() and
**update_model!**()) to perform Agents Based Modelling simulations.

```julia
function run!(model,update_agent!,update_model!)
    n = 0          
    while ( n < maxCycles)
        for agent in allagents(model)            
        update_agent!(agent)            
    end                                                    
    update_model!(model)                      
    n += 1                                                
    end                                                                 
end                                                                            
```

Figure 1


<table>
  <tr>
    <td align="centre" colspan="2">

    function run!(model,update_agent!,update_model!)
        n = 0          
        while ( n < maxCycles)
            for cell in allagents(model)            
                update_cell!(agent) 
            end           
        end                                                    
        update_model!(model)                      
        n += 1                                                
    end                                                                          

   </td>

   </tr>

   <tr>
   <td>

    function update_cell!(agent, model)
        updateIntegration
        executeCellEvents(model,agent)
    end         
   
   </td>

   <td>

    function() update_model!(model)
       model.nSteps += 1           
      executeModelEvents(model)                              
    end                          

   </td>

  </tr>

  <tr>

  <td>

    function executeCellEvents(model,cell)
        eventList = getfield(cell,:events_)
        for event in eventList
            if(event.test(model,cell,event))
                event.execute(model,cell,event)
            end 
        end                                               
    end                                                          

  </td>

  <td>

      function executeModelEvents(model)
           cell = nothing
           eventList = model.events
            for event in eventList 
               if(event.test(model,cell,event))
                   event.execute(model,cell,event)
                end
            end 
        end

  </td>

  </tr>
</table>

Figure 2

The basic Agents *run!*() function has been extended to allow users to
define an Agents model as a set of cells. Users control how cells behave
by attaching a set of 'Events' to each cell. An 'Event' is a Julia
struct consisting of a set of customised data and functions which
perform all of the required tasks for each Event.


```julia
mutable struct Event <: AbstractEvent
name_::String                      
name_::String                      
  functions_::Dict{Symbol,Any}
global_::Bool                        
end                                                      
```

Figure 2a Event is a predefined Julist struct




Each **Event** has a unique name, a Dictionary (for storing all required
data) and a set of functions for performing tasks associated with the
Event. The 'global\_' variable is set to 'true' if an Event is shared
between several cells. If global\_ is false, each cell will receive it's
own copy of the Event. Global Events can be used to save memory when
performing generic tasks, e.g. writing variables to an output file.
There are three functions which must be defined:test(), execute(),
save(). By default, test() is assigned a dummy function (which always
returns true), and save() is assigned a dummy function which does
nothing. The execute() function [must]{.underline} be defined. When the
program is running the test() function for that Event is always
executed. If test() returns true, the execute() function is called. The
save() function is used to save data (e.g. write variables to a file).
Events provide a simple mechanism which allows users to customise a
simulation by choosing different events, Figure 2. This mechanism also
allows programmers to add functionality by creating new Events.

<table>
  <tr>
    <td>

    function x()
    end

    </td>

    <td>

    function y()
    end

    </td>

  </tr>
</table>

Table

<img src = "./manual_fig2b.png" />


<table>
  <tr>
    <td colspan="2">
    </td>
  </tr>
</table>

An example of two functions, timeToDivide() and divide_cell() which can be used to create an Event which decides when a cell should divide and then creates two new daughter cells. The variables, nutrient and fraction, are stored in the Event data Dict. They are updated, when necessary, by the Event.

<img src = "./mainInputFile.png" />



The main input file uses a set of keywords:

**Cell**, **CellEvents**, **ModelEvents**, **Events**, **Initialise**,
**u0** , **Write**.

to define all of the required data.

**1) Define ODEs**

There are two mechanisms for creating ODEs

**a) Catalyst Reaction Network**

**Cell**:A cell_cat.dat

This command defines the input for creating ODEs using Catalyst

Format: Cell:\<*cell-label*\> \<*file*\>

Cell should be followed immediately by a colon and a Cell 'label', and
then a data file which defines the reaction network in Catalyst format
(see <https://github.com/SciML/Catalyst.jl>)

The format for the Catalyst data is defined in Appendix 1a.

The cell *label* must be unique because it allows users to define
multiple cell types which have different Catalyst reactions.

**b) Defining Customised ODE functions**

**Cell:**A cell_ode.dat

This command defines the input for creating ODEs using customised Julia
functions.

The functions must be defined in the file odeFunctions.jl

The format for ODEs functions is define in Appendix 1b.

**2. Defining Cell Events**

**CellEvents**:cell_events.dat

This command allows a user to define a set of Events which can be
attached to any cell.

Format *CellEvents*:\<f*ilename*\>

The key word, *CellEvents*, should be followed immediately by a colon,
and the name of the file containing the definitions for each CellEvent.
The format of this file is defined in Appendix 2.

The Cell Events file is used to create all cell Events dynamically (at
run time) by using the data and functions defined in this file to create
a set of Events which are then stored in a Dict.

**3. Defining Model Events**

**ModelEvents**:model_events.dat

This command allows users to define a list of all required model Events.

Format *ModelEvents:*\<f*ilename*\>

The key word, *ModelEvents*, must be followed by a colon, and then the
name of the file which contains definitions for a set of ModelEvents.
The format of this file is defined in Appendix 3.

The ModelEvents file is used to create all model Events dynamically (at
run time) by using the data and functions defined in this file to create
a set of Events which are then stored in a Dict.

**4. Creating Instances of Cell Events**

An Event *Instance* is an Event which has been created by copying a
predefined Event from the Events Dict and then modifying it by resetting
some of the variables stored in the Event data Dict. This allows users
to define customised Events at run time.

The 'Events' keyword is used to create instances of Events.

**Events**:A updateGrowthA{updateGrowth(growthLimit=1.0,fraction=0.65)}

Format Events:\<*cell label*\>
\<*instance-name*\>**{**\<*event-name*\>**(**comma separated
variables**)}**

The Events keyword allows users to assign specific instances of Events
to a set of cells with the same \<cell label\>.

This keyword is followed by a colon and then a cell label (e.g. A).

In the example above the \<*instance-name*\> is given a (unique)
name:*updateGrowthA*.

The remainder of the input enclosed by brackets, **{}**, defines an
instance of the Event, e.g.

{updateGrowth(growthLimit=1.0,fraction=0.65)}

This must correspond to a predefined Event (e.g. updateGrowth,
previously defined in the CellEvents file; see above).

This is subsequently parsed by removing the brackets, {}.

The remainder,

updateGrowth(growthLimit=1.0,fraction=0.65)

defines a Cell Event name, *updateGrowth*,

and a set of customised values for the Event variables:
(*growthLimit*=1.0,*fraction*=0.65).

The variable names and types must also be consistent with the original
definition of the Event.

An instance of this Event will be created at run time and assigned to
those cells which have the correct \<*cell-label*\>

**5. Creating Instances of Model Events**

Instances of Model Events can also be created using the Events keyword

Events:model update_nutrient{updateNutrient()}.

The only difference between this and the previous 'Cell Event' example
is the cell label has been replaced by 'model'. In this example, the
program will create an instance (labelled update_nutrient) of the
updateNutrient Event at runtime and attach it to the model.

**6. Initialising Cells**

Initialise:demo2.cells.dat

This command is used to define the initial set of cells.

Format *Initialise*:\<*cell-file*\>

\<*cell-file*\> contains the initial set of cells.

The format of the cell file is:

Cell \<*cell-label*\> \<*cell-id*\> (xccord, ycoord).

\<*cell-id*\> is a unique integer

e.g. Cell A 1 (x,y)

Cells defined in this file are used to build the ABM model. First a
model is created. Next the cells defined in this file are added. Each
cell is given a copy of the relevant CellEvent instances. Then the
ModelEvent instances are attached to the model. A serialized model is
then written to a .jls file.

**7. Defining Output Variables**

The **Write** keyword is used to define a list of variables to output,
e.g.

Write:Cell A rmr em rmq rmt et rmm mt mm q si mq mr r a rep mrep rmrep

Format Write:Cell \<*cell-label*\>
\<*list-of-space-separated-variables*\>

**Running a simulation**

run.jl will run a simulation

julia run.jl

will print out a *help* menu.

Running a simulation requires two steps

i\) julia run.jl -build \<*input-file*\>

e.g. julia run.jl -build demo.dat

reads the main data input file (demo.dat) and creates a serialised model
which is then written to a file (demo.js)

ii\) julia -model \<serialized-model-file\> -n \<int\>

e.g. julia run.jl -model demo.jls -n 200

will read a serialized model file (demo.jls) and run a simulation for n
cycles

**Appendix 1a**

Example input file for Catalyst reactions

<img src = "./appendix1a.png" />

The input file uses three keywords (**catalyst:,p,u0**)

**p** defines a set of p values

p is immediately followed by a space and then a set of p-values:
\<*name*\>=\<*value*\> (no spaces)

The defined p-values are separated by spaces.

**u0** defines initial values for u0, the values are defined as above.

**catalyst:** specifies the set of reactions (as specified in the
Catalyst documentation)

<img src = "./appendix1b.png" />


**Appendix 1b**

Example input for defining customised Julia ODE functions.

The keyword, *odeFunction*, (followed by colon) specifies a function
defining ODEs (e.g. *testFunction!*) which must be defined in
odeFunctions.jl

<img src = "./appendix1_ode.png" />

**Appendix 2**

Example ‘Cell’ Events file

<img src = "./appendix2.png" />

The first line is always *Events*, and the last line is always *end*.

The input for each is Event is controlled by several keywords.

**CellEvent**, **data**, **cell**, **model**, **equation**, **test**,
**execute**, **save**.

Keywords are always followed by a colon.

**CellEvent**:updateGrowth

Format *CellEvent*:\<*event-name*\>

*CellEvent* is followed by a colon and then a name for the Event. The
name must be unique.

**data**:growthLimit(Float64) = 1.0

The data keyword is used to define 'Event' data

Format data:\<*variable name*\>**(**\<*variable type*\>**)** =
\<*value*\>

\<*variable name*\> must be unique and \<*value*\> must correspond to a
valid \<*variable type*\>

Event data are stored in the Event data dictionary.

**cell**:growthProg(Float64) = 0.0

Format cell:\<*variable name*\>**(**\<*variable type*\>**)** =
\<*value*\>

The cell keyword is similar to the data keyword; the format is
identical.

The input is parsed and a new variable is attached to the cell.

This allows users to create their own cell variables.

**equation:**growthRateEqn = ((c_q + c_m + c_t + c_r) \*
(γ_max\*a/(K_γ + a)))/M

The *equation* keyword allows users to create their own equations, which
are then accessible to any function associated with the Event.

Format *equation*:\<*equation name*\> **=**\<*equation definition*\>

Equations are parsed and stored in the Event data dictionary.

The equation name must correspond to an equation required by at least
one of the Event functions. Functions which use this equation will
retrieve it, by name, from the data dictionary and evaluate it.

**reset:**growthProg=0.0, fraction=0.5

The *reset*: keyword is used to re-initialise events. This is most
likely to happen during cell division. When a cell divides each daughter
cell gets a copy of the parent's events. Some of these events may
require reinitialisation. In this example a CellEvent is monitoring the
growth of the current (parent) cell and the value of *growthProg* is
constantly changing. When the parent cell divides each daughter cell
inherits a list of Events from the parent cell. Many of the Event
variables should be reset. The default reset values for all the Event
variables are the initial values given to each variable. The reset:
keyword allows users to specify alternative values. In this example the
reset: keyword is used to define new initial values for *growthProg* and
*fraction*.

reset:growthProg = 0.0,fraction=0.5

Now, when the parent cell divides, *growthProg* will be set to zero and
*fraction* to 0.7

Format *reset*: \<v*ariable name 1*\> = \<*value 1*\> , \<*variable name
2*\> = \<*value 2*\>

Variables are always separated by commas.

**test**:checkGrowth

Format *test*:\<f*unction name*\>

test, defines the function an Event will use as the test function. The
function name must correspond to an existing function. By default, a
dummy function which always returns 'true' is used if test has not been
defined.

**execute**:divideByGrowth

Format *execute*:\<*function name*\>

execute, defines the function an Event will execute (if test() is true).

**save:**dontSave

Format save:\<*function name*\>

save, defines the function an Event will use to save data. By default, a
dummy function which does nothing is used if save has not been defined.

**Appendix 3** 

Example ‘Model’ Events file

<img src = "./appendix3.png" />
The first line is always 'Events' and the last line is always 'end'. The
format of this file is the same as appendix 2. The same keywords are
used. This example also uses the 'model' keyword.

**model**:nutrient(Float64) = 1000.0

This creates a new variable, *nutrient*, which is attached to the model.
In the above example the execute function is assigned to the function,
*updateNutrient*().



