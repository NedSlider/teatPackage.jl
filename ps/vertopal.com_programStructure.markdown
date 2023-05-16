

**[Program Design]{.underline}**

<p class = "aligncenter">
</p>
<img src="./ps_fig1.png"   class="center"/>

The program uses the *Agents* package
([*https://juliadynamics.github.io/Agents.jl/stable/*](https://juliadynamics.github.io/Agents.jl/stable/))
to simulate collections of cells. The main routine (*run!*) requires two
customised functions, *update_cell!* and *update_model!*, to perform an
ABM (Agents Based Model) simulation. To improve the overall flexibility
the current design uses 'Events' which allow users and programmers to
customise the behaviour of the software. Both *update_cell!* and
*update_model!* partition the simulation into a series of self contained
customised 'Events' by calling *executeCellEvents()* and
*executeModelEvents()* respectively.

<img src="./ps_fig2.png"   class="center"/>

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
CellData (struct which defines each cell) and model, Figure 3. This
allows a user to customise both models and cells. Programmers can also
use this feature to create new (more complex) Events which can
communicate with each other.

<img src="./ps_fig3.png"   class="center"/>

**Building A Model**

<img src="./ps_fig4.png"   class="center"/>

An example input file is shown above (the User Manual describes the
input in more detail). Most of the code is responsible for parsing a set
of input files and generating the model, set of cells and Events shown
in Figure 1.

<img src="./ps_fig5.png"   class="center"/>


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

<img src="./ps_fig6.png"   class="center"/>

**Appendix A:** Example of an Event which divides a Cell based on the
amount of nutrient consumed.

**Appendix B: The 'Cell' command**

Format Cell:\<*cell-label*\> \<*input-file*\>

e.g. Cell:A cell_ode.dat

Example data file

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

The function, parseEvent\_(), uses a set of keywords (*CellEvent, data,
cell, model, reset, equation, test,execute,save*) to parse the data
which is then returned as an Event. The User Manual explains what each
keyword does.

parseEvents\_() is called by readEventsFile() which appends each parsed
Event to a Dict{\<*event-name*\>,Event}.

**Appendix D: The Events Command**

Format **Events:**\<*cell-label*\>
\<*instance-name*\>**{**\<*Event-name*\>**(**\<*comma-separated-variables***)}**
