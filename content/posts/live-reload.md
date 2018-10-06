---
title: "Live-Reload in Fission: Instant feedback on your Serverless Functions"
date: 2018-10-06T01:46:30-07:00
draft: true
---

Fast feedback loops are an important devops principle: the sooner you
find a bug, the cheaper it is to fix it.

While coding, you're typically going through a cycle -- write code,
build, deploy into a test environment, run some test, fix, repeat.
The build and deploy parts of this cycle are unproductive time for the
developer.  Once these steps are slower than a few minutes, you'll end
up context switching to another task; and that means if you discover a
bug in test, it's more cumbersome to get your context back and debug
it.

With live-reload in Fission, we've brought the time from code to
running test down to a few seconds.

## Using live-reload in Fission

The `fission` CLI supports [declarative specifications](...) for
Fission resources -- packages, functions, etc.  These specifications
also configure packaging -- how source code is bundled up and uploaded
to the cluster.  They're stored as a set of YAML files in a `specs`
directory at the root of your Fission application.  You don't have to
write these YAML files from scratch; Fission generates initial
versions for you.

The deployment of all resources in the specifications is triggered by
the `fission spec apply` command.  With this command, the fission CLI
checks the state of the cluster against the desired state as
configured in the specs.

To use live-reload, simply add the --watch option:

```sh
$ fission spec apply --watch
```

And then, edit your code in your editor as usual.  Whenever you save a
file, the fission CLI notices a change on the filesystem, and packages
up the code and uploads it.  On the cluster, if this is a compiled
language, the source is compiled. Then the function deployment is
automatically updated.  Within a few seconds, the new function is
ready to be tested.

Here's a quick demo video of live-reload in action:

<script src="https://fast.wistia.com/embed/medias/vq3p38cetv.jsonp" async></script><script src="https://fast.wistia.com/assets/external/E-v1.js" async></script><div class="wistia_responsive_padding" style="padding:60.94% 0 0 0;position:relative;"><div class="wistia_responsive_wrapper" style="height:100%;left:0;position:absolute;top:0;width:100%;"><div class="wistia_embed wistia_async_vq3p38cetv videoFoam=true" style="height:100%;position:relative;width:100%"><div class="wistia_swatch" style="height:100%;left:0;opacity:0;overflow:hidden;position:absolute;top:0;transition:opacity 200ms;width:100%;"><img src="https://fast.wistia.com/embed/medias/vq3p38cetv/swatch" style="filter:blur(5px);height:100%;object-fit:contain;width:100%;" alt="" onload="this.parentNode.style.opacity=1;" /></div></div></div></div>

## Behind the scenes -- how it works

picture:

[editor] -> `fission spec apply --watch` -> [fission storagesvc] -> [builder] -> [back to storage] -> [runtime pod]

## Conclusion

Live reload in Fission gives you a smooth, fast development
experience.

Check out the [Fission installation guide](https://docs.fission.io/installation/) to get started with
Fission.  Join us on the [Fission Slack](http://slack.fission.io) to chat, or follow us on Twitter at @fissionio.
