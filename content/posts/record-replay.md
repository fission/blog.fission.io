---
title: "Record-replay in Fission"
date: 2018-10-06T01:46:30-07:00
draft: true
---

Functions in a Fission are triggered by events.  We've recently added
a new feature to Fission that allows you to _record_ these events into
a database, examine these recordings, and replay them.

Functions are often part of a larger, more complex system, that can
take a while to test.  Or, tests might not be automated and require
some human interaction.  In both cases, you can record the events that
trigger the function and simply use the same events to re-test the
function.  The recording can be replayed on different versions of a
function, allowing to regression test ("did that change break
anything?") or test bugfixes in new versions ("did that change fix the
bug?")

## Using record-replay in Fission

Record replay is available from Fission 0.11 onwards.  You can enable
recording for a function or a trigger.  This section will guide you
through using record replay on a simple function in Python.

First, let's download a sample function:

```
$ curl -LO https://xxx/hi.py

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

Now comes the fun part -- let's enable recording for this function:

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

pictures:

without recording:

```
request -> fission router -> function
```

with recording:
```
request -> fission router -> function
              -> redis
```

replay:
```
replay request -> fission replay api -> router -> function
                     <-> redis
```

## Conclusion

Record replay in Fission gives you a powerful, state of the art tool
to develop and test your functions.

Check out the [Fission installation guide](https://docs.fission.io/installation/) to get started with
Fission.  Join us on the [Fission Slack](http://slack.fission.io) to chat, or follow us on Twitter at @fissionio.
