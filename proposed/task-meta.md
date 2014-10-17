  
Task Meta Document
==========================

1. Summary
----------

Almost every framework includes a toolset for running background tasks, cron jobs, deployment and more.
These usually provide a set of predefined tasks that are configured and combined to achieve some specific goal.
Common examples of such tasks would be copying a file on the filesystem, executing a database query, etc.
Each such library provides its own set of guidelines on how new tasks should be added to its disposal,
thus forcing the developer to creating multiple adapters or supporting only a limited subset of libraries.

2. Why Bother?
--------------

This proposal presents a simple API that task runner libraries might expect the tasks to implement.
It will also attempt to make these tasks easily portable and pluggable, but this is not the primary goal.

3. Scope
--------

## 3.1 Goals

* Provide the interface for runnable tasks
* Keep the interfaces as minimal as possible.
* Attempt at achieving a pluggable interface allowing for easier integration. This is not the primary goal though.
* Task Runner Examples: Bldr, TaskPHP, Robo, Phing

## 3.2 Non-Goals

* This proposal does not define any rules for how the tasks are to be configured or grouped into jobs.
* Examples of things which this PSR does not cover: Symfony2 commands, Yii commands

4. Design Decisions
-------------------

## Task execution

It is obvious that tasks should be runnable, hence a TaskInterface::run() method is proposed.

## Output

To show progress to the user tasks need to have a way to output information. These are different ways of achieving this that have been considered:

* **Standard output** using php ouput functions like echo. This is the most simple approach, but it relies on a global output stream and thus limits areas in which it can be applied. For example if your task runner library should email you with the whole task log after the job is complete it would have to rely on output buffering to catch the output.
* **Returning a message** string. This method avoids the pitfals of the previous one but since the result is returned at the end of the function call this approach would strip all of the interactivity of the process. Especially for tasks that would take a long time to run and would have a lot of intermediatery messages like: "Connecting to host", "Logging in", "Running command", "Cleaning up". Ideally these messages should be delivered instantly to the user instead at the end in bulk.
* **Yielding messages** line by line. A slightly modified version of the ebove would yield message lines one by one. This would allow for interactivity, but is not available in older PHP versions. This approach would not allow appending data to the current line if implemented like that.
* **Output interface** with functions like write() and writeln() seems the both the easiest to understand and implement. That's why it is currently chosen.

## Output formatting

A lot of times we would like to add some formatting to task output, like nicely formatting a table. This would be possible by adding relevant methods to the OutputInterface. Ultimately after looking at some existing tasks in popular libraries it seems that these aren't used that often. In that case it is better to allow tasks to do formatting themselves. Especially since it's very easy for a Task to depend on some string formatter using DI and the write that strig to the OutputInterface.

## Configuration

Most of the tasks require at least some parameters to operate ( e.g. FTP credentials, source directory and destination directory for a task dealing with FTP uploads). Here are the considered approaches to pass these parameters to a Task:

* **As an array parameter to run()** representing a key=>value pairs of options. In the event that the task would need to be run twice with slightly different parameters ( e.g. uploading two directories to an FTP server one by one ) such an approach would require duplicate parameters to be present both times it is ran ( FTP login and password would have to be passed for both first and second call ).
* **Using setParameter($key, $value)** would allow overriding single parameters, and thus solve the issue described above.

## Error handling

Suggested ways of error handling were:

* **Using return codes** for errors, similar to CLI apps. An integer error carries much less information to the user than a string message. A user might of course first output the error through OutputInterface and then return an error code. But then there would be no way to distinguish between regular messages in the outout interface and error messages. Which might be desireable for for example formatting ( a task runner might like to output errors in a different color or to a different stream). Each talk would have tto have a try-catch block to return an error code when an exception happens.
* **Using exceptions for errors** seems the most logical, as it clearly tells the error message to the task library that ran the test. This approach doesn't require a try-catch black inside the task code. The running library can easily wrap calls to run() inside such a block

## Parameter validation

The need for a separate validate() method was debated. Such a method would check whether currently set parameters are sufficient and valid. But since a task would have to run validate() internally as the first step of run() to check that parameters are correct it seems there is no need for this method to be called twice. So parameter validation should be done inside run() and throw an exception if something is wrong with them.

5. Implementation in popular libraries
--------------------------------------

This section describes steps that would have to be taken to make popular libraries comply with this PSR.
The purpose is to show how easily this could be done.

### Phing

 * Task::main() would have to be renamed to Task::run, or ::run could act as a proxy for ::main for backwards compatibility
 * When setting configuration values instead of setSourceDir('/') setProperty('sourceDir', '/') would be called

### Bldr

Is mostly compliant anyway. All it needs is for CallInterface to extend TaskInterface, and some tweaks with passing parameters.
There is no need to change internal nomenclature and rename CallInterface to TaskInterface internally.

### Robo

Robos' TaskInterface already defines ::run(), but would be required to also pass OutputInterface to it, instead of relying on static Runner::getPrinter() call

### Task

Realies on Symfony2 Command so most of the things i said about Symfony 2 apply here too.




