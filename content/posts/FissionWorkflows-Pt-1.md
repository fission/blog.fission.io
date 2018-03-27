---
title: "Fission Workflows: Part 1"
date: 2018-03-27T04:25:55-07:00
draft: true
---


## Open Source Serverless Workflows for Kubernetes
Fission provides fast serverless functions on Kubernetes. While functions are great for specific pieces of business logic, any non-trivial application requires a composition of functions.

There are many ways to compose functions. You can directly call functions from each other — but there are some disadvantages to this. For one, the structure of the application becomes hard to understand; dependencies are not obvious; essentially, every function becomes an API. Second, there’s no persistent state; if there’s a failure or exception and you want to retry, the whole function must run again.  

Alternatively, you can wire up a set of functions using message queues to send the output of one function to a message topic that triggers another function. While this helps with persistence and retries (because you can retry individual functions rather than the whole composition), it doesn’t help at all with making the system easier to understand. It’s also hard to replicate any dynamic control flow (such as an if statement or a loop).


## Workflows
A third way to compose functions is using workflows.Think of a flowchart: a sequence of tasks, decisions, loops and so on. A flowchart makes the structure of a complex task obvious. 

Workflows are like flowcharts for serverless functions, except they’re more powerful. You can compose together functions in sequence or in parallel, send the output of a function to the inputs of another, write if statements, loops, and even functions that operate on other functions.

### Use cases, Demos, Examples

Broadly we see a few areas where workflows are useful:

- Devops automation use cases
- Data processing use cases
- Business process automation


### Demos









## Terminology/Concepts

Here are a few terms and concepts that you should know in order to better understand workflows.

A _**Task**_ is a unit of computation within a workflow. Most commonly, it is a function call. It takes in data input and outputs the resulting data or error. A task can specify its input using a Javascript expression; this allows inline transformations of data from the output of other tasks. A workflow is defined as a list of tasks.

_A task can have a **dependency** on one or more other tasks. With dependencies you can manipulate the control flow within a workflow._

A _**Dynamic Task**_ can manipulate control flow. For example, it could be an if-statement, a loop, etc. A dynamic task is just a regular task that returns another task instead of data. Users can extend workflows by writing their own dynamic tasks.

_Control flow constructs_ are available as dynamic tasks. For example, an “if” dynamic task checks the value of an expression and executes one of two tasks given to it.

Workflows themselves are Fission functions — this allows workflows to be triggered the same way as any other function.

Workflows are specified in _**YAML**_. 

















