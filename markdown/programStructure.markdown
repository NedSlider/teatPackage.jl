

**[Program Design]{.underline}**

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


Figure 1


The program uses the *Agents* package
([*https://juliadynamics.github.io/Agents.jl/stable/*](https://juliadynamics.github.io/Agents.jl/stable/))
to simulate collections of cells. The main routine (*run!*), Figure 1, requires two
customised functions, *update_cell!* and *update_model!*, to perform an
ABM (Agents Based Model) simulation. To improve the overall flexibility
the current design uses 'Events' which allow users and programmers to
customise the behaviour of the software. Both *update_cell!* and
*update_model!* partition the simulation into a series of self contained
customised 'Events' by calling *executeCellEvents()* and
*executeModelEvents()* respectively.


```julia
mutable struct Event <: AbstractEvent
name_::String                      
name_::String                      
  functions_::Dict{Symbol,Any}
global_::Bool                        
end                                                      
```
Figure 2

An Event is a Julia struct, Figure 2. Each Event must have a unique
name. An Event also has a data Dict for storing all required data (e.g.
variables). There is another Dict for storing the (customised) functions
associated with each Event. Three functions must be
pre-defined:*test()*, *execute()* and *save()*. By default, test() is
assigned a dummy function which always returns true. The save() function
is used to save data (e.g. write variables to a file). By default,
save() is assigned to a dummy function which does nothing. The execute()
function [must]{.underline} be defined. When an Event is executed (see
Figure1: *executeCellEvents,* *executeModelEvents*) the test() function
is called. If test() is true, the execute() function is called. This
design allows users to customise a simulation by a selecting a specific
set of Events for each simulation. It also provides a generic interface
for adding new functionality. New Events and code (data,functions) can
be added without changing the overall design of the program, Figure 1.
All required Events must be predefined in an ascii file. Appendix A
contains an example of a Cell Event which divides a cell. Several
keywords (*data,cell,model,test,save,execute*) are used to create a new
Event (a detailed explanation of how to define Events is given in the
user manual).

The *data*, *cell*, or *model* keywords can be used to add variables.
E.g.

<data:fraction(Float64)=0.6>

adds the variable, *fraction*, to the Event data Dict. Alternatively,
the cell keyword can be used to attach a new variable to the cell, e.g.

cell[:fraction(Float64)=0.6](data:fraction(Float64)=0.6)

This creates a new *cell* variable, *fraction*. Similarly, the *model*
keyword can be used to create new model variables. This is an important
feature of the overall design, because Events can be used to customise
both the ABM model and individual cells by creating new cell and model
variables. This makes it easier to output data. If variables are defined
using cell or model, they are easier to manipulate (e.g. outputting a
set of cell variables can be handled by a new Event). Cell and model
variables can also be used by Events to communicate with each other.

The cell and model keywords enable Events to add new variables to
CellData (struct which defines each cell) and model. This
allows a user to customise both models and cells. Programmers can also
use this feature to create new (more complex) Events which can
communicate with each other.


**Building A Model**


  **Main Input File**
```
  Cell:A cell_a.dat                                               # Catalyst input data
  CellEvents:cell_events.dat                               # contains all predefined cell events
  ModelEvents:model_events.dat                       # contains all predefined model events
  Events:A updateGrowthA{updateGrowth(growthLimit=0.01,fraction=0.55)}
  Events:model updateNutrientA{updateNutrient()}
  Events:model saveCellData{saveData()}
  Events:model cellCount{modelCellCount()}   # events attached to model
  Initialise:demo.cells.dat                                    # contains the initial cells
  Output:./demo file-prefix
  u0:A steadystate.csv 1.0
  Write:Cell A r e_t e_m q m_r m_t m_m m_q c_r c_t c_m c_q a s_i
```

Figure 4

An example input file is shown above (the User Manual describes the
input in more detail). Most of the code is responsible for parsing a set
of input files and generating the model, set of cells and Events shown
in Figure 1.

<img src="./ps_fig5.png"/>

Figure 5


The function, *parseInputFile()*, parses the main input file and
performs the following tasks.

**1. Cell** (step 1)

This command is processed by the parseCell() function which returns a
'cell_params' struct, which is then stored in model.cellParamsMap.

In this example (Figure 4), the command

Cell:A cell_a.dat

parses a file (cell_a.dat) which defines the ODEs for all cells
labelled, 'A'.

The struct returned by parseCell() is stored in a Dict,
model.cellParamsMap, which is then used to construct the ODE integrator
for all cells labelled, 'A'. Appendix B explains how this command is
processed.

**2. CellEvents** (step 2)

The function, readEventsFile(), parses an ascii file which defines Cell
Events and appends each event to a Dict{name,Event}.

Appendix C explains how this command is processed.

**3. ModelEvents** (step 3)

This command is very similar to the CellEvents command (step 2). It also
uses the function, readEventsFile(), which parses the Model Events and
creates a Dict{\<event-name\>,Event} which contains every predefined
Model Event.

**4. Events:A** (step 4)

This command defines Cell Event instances. Event Instances are used to
create the set of events which will be attached (dynamically) when each
cell is initialised, Figure 1.

The function, parseEventInstance\_\_(), is used to parse the data.

It returns a
tuple(\<*instance-name*\>,\<*event-name*\>,\<*variable-lis*t\>).

The tuple is stored in a Dict{\<i*nstance-name*\>,tuple}.

Appendix D explains how this data is parsed.

**5 Events:model** (step 5)

This command is similar to the previous Events command (step 4). It
generates a tuple defining a model instance which is stored in a Vector.

**6. Initialise** (step 6)

This command defines a file which contains the initial set of cells.
This is used to generate an initial list of cells. The User Manual
explains the format in more detail.

The function, initialiseModel(), uses the data generated in steps 1-7
(Figure 5) to build the model.

First an initial model is generated, and then the cells are added.

<img src="./ps_fig6.png"/>

Once the model has been built it is written out to a  serialized file (.jls).

**Appendix A:** Example of an Event which divides a Cell based on the
amount of nutrient consumed.

<table>
  <tr>
    <td>

      Cell Events are defined in a separate ascii file. 
      This is explained in detail in the User Manual. 
      Users define the data (variables)  and functions 
      required. In this example 3 variables have been 
      declared:nutrient, maxSteps, fraction. The test()
      function has been assigned to timeToDivide(). 
      When this returns true, divide_cell() is executed.
    
  </td>
  <td>

      CellEvent:divideByTime 
      data:nutrient(Float64) = 0.00005
      data:fraction(Float64) = 0.5
      test:timeToDivide
      execute:divide_cell
      save:do_nothing

  </td>
  </tr>

  <tr>
  <td>

      function timeToDivide(model,cell,event)
         eventData = getfield(event,:data_)
         nutrient = Events.getEventVariable("nutrient",eventData)
         nCells = length(model.agents)
         max = 0.05 - nCells * nutrient
         nSteps = getfield(cell,:nSteps_)
         probability = nSteps * rand(Uniform(0.0,max))
         if(probability > 1.0)
             return(true)
         end
         return(false)
     end

  </td>

  <td>

      function divide_cell(model,cell,event)
        model.lastCell += 2
        data = getfield(event,:data_)
        fraction = Events.getEventVariable("fraction",data)
        u1 = divideResources(cell,fraction)
        u2 = divideResources(cell,1.0-fraction)

        cellLineage = getfield(cell,:lineage_)
        cellLineage.status_ = "divided"
        cell1 = createNewCell(model,cell,nextAgent,u1,(0.0,0.0),cellIndex)
        cell2 = createNewCell(model,cell,nextAgent+1,u2,(0.0,0.0),cellIndex)
        kill_agent!(cell,model)
        return()
      end

  </td>
  </tr>
</table>



**Appendix B: The 'Cell' command**

Format Cell:\<*cell-label*\> \<*input-file*\>

e.g. Cell:A cell_ode.dat

Example data file

```
catalyst:catalyst.dat    # Catalyst reaction
p	s=1e4	d_m=0.1	n_s=0.5	n_r=7459	n_t=300	n_m=300	n_q=300	γ_max=1260	K_γ=7	v_t=726 K_t=1000	v_m=5800	K_m=1000	w_r_max=930	w_t_max=4.14	w_m_max=4.14	w_q_max=949	θ_r=427	θ_t=4.38	θ_m=4.38	θ_q=4.38	K_q=152219	h_q=4	kb=0.95e-2	ku=1	M=1e8
u0	r=10	e_t=0	e_m=0	q=0	m_r=0	m_t=0	m_m=0	m_q=0	c_r=0	c_t=0	c_m=0	c_q=0	a=1000	s_i=0
```

This example file defines a Catalyst reaction network. The data is
parsed by parseCell.jl.

There are three keywords: catalyst, p , u0.

The format for each keyword is defined in the User Manual.

The **catalyst** keyword, is followed by a colon and then a file which
contains a Catalyst reaction network. This is converted to a Catalyst
ReactionSystem

**p** defines a set of p-values

**u0** defines the u0 values

The data is return as a 'cell_params' struct.

**Appendix C: The CellEvents command**

Format CellEvents:\<*filename*\>

e.g. CellEvents:cell_events.dat

This command parses an ascii file containing a list of predefined Cell
Events.

This file is parsed by the function,readEventsFile(). Data for each
Event is extracted and parsed by the function, parseEvent\_().

Example input data.

```
Events
CellEvent:updateGrowth
data:growthLimit(Float64) = 1.0
data:fraction(Float64) = 0.6
data:growthIncrease(Float64) = 0.0
cell:growthProg(Float64) = 0.0
reset:growthProg = 0.0,growthIncrease = 0.0
equation:growthRateEqn = ((c_q + c_m + c_t + c_r) * (γ_max*a/(K_γ + a)))/M
test:checkGrowth
execute:divideByGrowth
save:dontSave
end
```

The function, parseEvent\_(), uses a set of keywords (*CellEvent, data,
cell, model, reset, equation, test,execute,save*) to parse the data
which is then returned as an Event. The User Manual explains what each
keyword does.

parseEvents\_() is called by readEventsFile() which appends each parsed
Event to a Dict{\<*event-name*\>,Event}.

**Appendix D: The Events Command**

Format **Events:**\<*cell-label*\>
\<*instance-name*\>**{**\<*Event-name*\>**(**\<*comma-separated-variables***)}**

e.g. Events:A updateGrowthA{updateGrowth(growthLimit=1.0,fraction=0.6)}


