The abstraction provided by the OS of a running program is something we will call a process.

Important things for the process:
1. Memory address Space.
2. Registers. i.e. PC(Program Counter)
3. Stack Pointer

There are several API's provided by the Modern OS for the process.

• Create: An operating system must include some method to create new processes. When you type a command into the shell, or double-click on an application icon, the OS is invoked to create a new process to run the program you have indicated. 

• Destroy: As there is an interface for process creation, systems also provide an interface to destroy processes forcefully. Of course, many processes will run and just exit by themselves when complete; when they don’t, however, the user may wish to kill them, and thus an interface to halt a runaway process is quite useful.

• Wait: Sometimes it is useful to wait for a process to stop running; thus some kind of waiting interface is often provided. 

• Miscellaneous Control: Other than killing or waiting for a process, there are sometimes other controls that are possible. For example, most operating systems provide some kind of method to suspend a process (stop it from running for a while) and then resume it (continue it running). 

• Status: There are usually interfaces to get some status information about a process as well, such as how long it has run for, or what state it is in.


There are multiple process that are running , when these process switch context there last state of registers must be store somewhere. So, these are stored in a data structure


CPU Utilization:

$$\text{CPU Utilization} = \frac{\text{Time CPU is busy}}{\text{Total Time}} \times 100$$