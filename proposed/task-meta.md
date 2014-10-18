  
Task Meta Document
==========================

1. Summary
----------

Automation tools allow developers to setup frequently used processes such as building, deployment and testing.
They feature sets of predefined tasks which are used as building blocks for process definition.
Examples of such building blocks might be copying a file on the filesystem, executing a database query, running a command on a remote server, etc. Since these are pretty common on across libraries this leads to redundancy in task implementation also forcing 3rd-party developers to create multiple adapters or supporting only a limited subset ofautomation tools.

2. Aim
--------------

This proposal aims to create a minimal API to allow task implementations to be reused in multiple automation toolsets.

3. Scope
--------

## 3.1 Goals

* Provide the interface for runnable tasks
* Keep the interfaces as minimal as possible.
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

## Injecting OutputInterface

There was discussion on whether OutputInterface should be injected separately or passed as a run() method parameter.
Making it a run parameter makes it clearly visible that the OutputInterface is not optional. 

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

