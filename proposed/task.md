Task interfaces
=======================

Automation tools allow developers to setup frequently used processes such as building, deployment and testing.
They feature sets of predefined tasks which are used as building blocks for process definition.
Examples of such building blocks might be copying a file on the filesystem, executing a database query, running a command on a remote server, etc. Since these are pretty common on across libraries this leads to redundancy in task implementation also forcing 3rd-party developers to create multiple adapters or supporting only a limited subset ofautomation tools.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119].

[RFC 2119]: http://www.ietf.org/rfc/rfc2119.txt

## Goal

This proposal aims to create a minimal API to allow task implementations to be reused in multiple automation toolsets.

## Specification

A task is an atomic optionally configurable action. Automation libraries provide means for combining and configuring these tasks to automate processes.

## Interfaces

### OutputInterface

OutputInterface defines a minimal output stream that MUST be used for writing messages by a task. If the task requires additional formatting capabilities it SHOULD handle those itself or make use of some injected formatter.

```php
<?php

namespace Psr\Task;

/**
 * Output stream for writing messages
 */
interface OutputInterface
{
    /**
     * Writes a message to the output.
     *
     * @param string $message The message
     *
     * @return void
     */
    function write($message);

    /**
     * Writes a message to the output and adds a newline at the end.
     *
     * @param string $message The message
     *
     * @return void
     */
    function writeln($message);

}
```

### TaskInterface

TaskInterface defines an API for configuring and running tasks. Paramter validation MUST be triggered internally by the run() method. Additionaly a task MAY validate parameters as they are passed to setParameter.

```php
<?php

namespace Psr\Task;

/**
 * A specific executable action
 */
interface TaskInterface
{
    /**
     * Executes the task.
     *
     * @param OutputInterface $output Output stream
     *
     * @return void
     */
    public function run(OutputInterface $output);
    
    /**
     * Sets a configuration parameter.
     *
     * @param string $name  Name of the configuration parameter
     * @param mixed  $value Parameter value
     *
     * @return void
     */
    public function setParameter($name, $value);

}
```
