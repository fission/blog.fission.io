---
title: "Record-replay in Fission"
date: 2018-10-16T01:01:00-07:00
publishDate: 2018-10-15T01:04:00-07:00
draft: false
---

Fission Functions are triggered by events. We’ve recently added a new
feature that allows you to record these events into a database,
examine these recordings, and replay them for testing and
troubleshooting.

Functions are often part of a larger, more complex system, that can
take a while to test.  Sometimes, tests might not be fully automated
and require some human interaction.  In both cases, you can record the
events that trigger the function and simply use the same events to
re-test the function.  The recorded events can be replayed on
different versions of a function, essentially functioning as an
automated test.

Recording can also useful in production -- for example, you can enable
recording for a subset of function invocations that error out, and use
those recorded events later to test bugfixes.

Fission is the first serverless function framework to offer out-of-the
box record-replay.

## Using record-replay in Fission

Record replay is available from Fission 0.11 onwards.  You can enable
recording for a function or a trigger.  This section will guide you
through using record replay on a simple function in Python.

First, let's download a sample function:

```
$ curl -LO https:// /hi.py

# This function is a hello world in Python.
$ cat hi.py
from flask import request
from flask import current_app

def main():
   name = request.args["name"]
   return "Hello, %s" % name
```

Next we'll create the function and trigger on the cluster:

```
# Ensure we have the Python environment set up for our cluster
$ fission env create --name python --image fission/python-env

# Create function
$ fission function create --name hi --env python --code hi.py

# Create HTTP trigger
$ fission route create --function hi --url hi --method GET

```

**Now comes the fun part -- let's enable recording for this function:**

```
$ fission recorder create --name my-recorder --function hi
```

Next, send some requests to the function.  These requests will be
recorded.

```
$ fission function test --name hi --query name=World
Hello, world!

$ fission function test --name hi --query name=Kubernetes
Hello, Kubernetes!
```

You can view the recorded events:

```
$ fission records view -v
REQUID                                  REQUEST METHOD FUNCTION RESPONSE STATUS TRIGGER
REQ425b0490-ef6b-42e5-b364-036b77250d8c GET            hi       200 OK          c26107ea-8e4d-42fe-a0b0-23c67a4ff669
REQc21596dc-75a2-4542-ac20-45d3f30dfa75 GET            hi       200 OK          c26107ea-8e4d-42fe-a0b0-23c67a4ff669

```

And you can choose an event to replay:

```
$ fission replay --reqUID <UID>
Hello, world!
```

## A quick overview of how record-replay works

The Fission installation includes a Redis deployment for
record-replay.  (You can also switch this off and install without
Redis if you're not planning to use record-replay.)

[image]

## Conclusion

Record replay in Fission gives you a powerful, state of the art tool
to develop and test your functions.

Check out the [Fission installation guide](https://docs.fission.io/installation/) to get started with
Fission.  Join us on the [Fission Slack](http://slack.fission.io) to chat, or follow us on Twitter at @fissionio.
