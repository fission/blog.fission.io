---
title: "Automated Canary Deployments in Fission"
date: 2018-10-06T01:46:30-07:00
draft: false
---

Canary Deployments are a time-tested deployment strategy.  The
fundamental idea is that deploying software into a production cluster
is different from _releasing it to its users_.  With canary
deployments, you deploy both old and new versions into a production
environment, but send only a small percentage of traffic to the newer
version.  That way, if the new version fails, only a few users are
affected rather than the application's entire user base.  If the newer
version works well, the traffic proportion being sent to it is
increased further.

With _Automated Canary Deployments_, the feedback loop is automated --
success causes the new version's proportion of traffic to be
increased, and failue causes the new version's traffic proportion to
be rolled back to zero.

Fission features Automated Canary Deployments for Functions.  Below,
we'll go over a quick tutorial on using this feature in Fission.


## Using Automated Canary Deployments in Fission

Automated Canaries are easy to use in Fission.  You'll need Fission
0.11 or newer.

For this tutorial, we'll start with two versions of a function
deployed on a cluster, and show how automated canary deployments
incrementally transfer load from one version to another.

First, download a couple of trivial sample functions:

```
$ curl
$ curl
```

Next, create the environment and functions on your Fission cluster:

```
$ fission env create --name nodejs --image fission/node-env

$ fission fn create --name func-v1 --env nodejs --code func-v1.js

$ fission fn create --name func-v2 --env nodejs --code func-v2.js
```

Next, create a route with two weighted functions.  This route is set
up to send 100% of load to `func-v1`, and none to `func-v2`.  Still,
it's important that it's set up for two function versions.

```
$ fission route create --name route-canary \
       --method GET --url /canary          \
       --function func-v1 --weight 100     \
       --function func-v2 --weight 0 
```

Fission's canary automation can roll out the newer version by changing
these weights over time.  Let's create a canary configuration
resource, which configures the parameters of how exactly the automated
rollout will occur, and when it will rollback if necessary.

```
fission canary-config create --name canary-1        \
       --funcN func-v2 --funcN-1 func-v1            \
       --httptrigger route-canary                   \
       --increment-step 10 --increment-interval 30s \
       --failure-threshold 10
```

First, the configuration needs to know the old and new versions.
Next, it needs to know which HTTP route to canary.  The next two
parameters, step and interval, specify how much the traffic shifts and
how often; in other words it's the rate of progress for the canary.
Finally, the user specifies the maximum permissible percentage of
failures; failures beyond this percentage cause a roll back.

The rest is automatic.  In this example we've set the roll out rate to
be pretty fast.  In reality you'd probably want have it much slower,
so you have more real-world testing on the new version before
incrementing it to significant percentages.

With Prometheus, you can view the traffic to the old function drop as
the traffic to the new version rises:

[prometheus graph screenshot]

## An overview of how Automated Canary Deployments work

[use the diagram from Smruthi's slides]

## Conclusion

Automated Canary Deployments in Fission give you the ability to move
changes into production with lower risk and increased confidence.

Check out the [Fission installation guide](https://docs.fission.io/installation/) to get started with
Fission.  Join us on the [Fission Slack](http://slack.fission.io) to
chat, or follow us on Twitter at @fissionio.
