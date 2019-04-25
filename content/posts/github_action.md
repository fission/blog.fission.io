---
title: "CI/CD with Fission GitHub Action"
date: 2019-04-25T18:30:53+05:30
draft: false
author: "[Vishal Biyani](https://twitter.com/vishal_biyani)"
---


# Introduction

GitHub recently launched [GitHub actions](https://github.com/features/actions) which enables developers to develop workflows and execute them based on events in code repositories such as a push event or an issue creation. There are many actions available on [the Github marketplace](https://github.com/marketplace?type=actions) which you can use for automating various tasks and workflows around development and deployment. In this tutorial we will use recently launched [Fission action](https://github.com/marketplace/actions/fission-action) and build a simple workflow to deploy a Fission function to a Kubernetes cluster.

# Prerequisites

You will need a Kubernetes cluster with Fission installed. Please follow [instructions here](https://docs.fission.io/installation/) for installing Fission in your Kubernetes cluster. After that verify that fission cli is working with fission by using the `fission --version` command.

# Local Development

Let's create a simple NodeJS function which prints "Hello World!" when invoked.

```
$ fission spec init
$ cat hello.js
module.exports = async function(context) {
    return {
        status: 200,
        body: "Hello, Fission!\n"
    };
}
$ fission env create --name node --image fission/node-env --spec
$ fission fn create --name hellonode --env node --code hello.js --spec
```

At this point we can apply the function specs and  test the function. We now have a working function which we can commit to a GitHub repository.

```
$ fission spec apply
$ fission fn test --name hellonode
Hello World!
```

# CD workflow with Fission action

The next step is to automate the deployment of function to a cluster on every push. For this we will use a Fission action based workflow. If you are interested in details of Fission action you can continue or you can skip to next sub-section of building workflow with Fission GitHub action.

## Understanding Fission GitHub Action

Fission GitHub action is a container which has a Kubectl CLI, Kubeconfig template & Fission CLI built into it. The default entrypoint populates the Kubeconfig template based on environment variables and then runs the `fission spec apply` command.


## Building workflow with Fission action

Now using this action - let's build a simple workflow which will apply functions if there is a push event. The workflow is defined in `.github/main.workflow` file in same repository as code. You can check out more details and options of GitHub workflow [here](https://developer.github.com/actions/managing-workflows/workflow-configuration-options/)

```
workflow "Fission CD" {
  on = "push"
  resolves = [
    "FissionCD",
  ]
}

action "FissionCD" {
  uses = "docker://fission/github-action:1.0.0"
  secrets = ["CERTIFICATE_AUTHORITY", "SERVER_ADDRESS", "USER_TOKEN"]
}

```

Push the code to a repository with workflow definition, assuming you have actions enabled in your repo, you will see an action tab. You will notice that workflow is marked as invalid and has errors. This is because it does not have values of the secret available yet.

![Workflow Error](../../images/githubaction/github_workflow_error.png)


Edit the workflow and enter the values of secrets required to execute the workflow. You can copy the value of certificate authority, token and Kubernetes server address from Kubeconfig for purpose of testing but ideally suggested to use a dedicated service account with appropriate access. GitHub makes these secrets available to the containers running the workflow as environment variables.


![Workflow Secrets](../../images/githubaction/workflow_secret.png)

Once you have entered all secret values - save and commit change to master. This will kick off the workflow and you will see the workflow running and you can also checkout logs:


![Workflow Secrets](../../images/githubaction/workflow_running.png)

In the logs, we can see that Fission action applied the specs and the environment and function was created as seen in logs below. You can test out the function now!

```
### STARTED FissionCD 15:16:50Z

Pulling image: gcr.io/github-actions-images/action-runner:latest
latest: Pulling from github-actions-images/action-runner
169185f82c45: Pulling fs layer
0ccde4b6b241: Pulling fs layer

..
..
..


8522a1fc57c5: Pull complete
Digest: sha256:43a18edb4167aca86e24424a985417a490e99a8b432aa75404e18cba56052176
Status: Downloaded newer image for vishalbiyani/fission-action:7
1 environment created: nodejs
1 package created: hello-js-cpdz
1 function created: nhello

### SUCCEEDED FissionCD 15:17:32Z (41.909s)
```


# There is more!

What we tried out in this tutorial is simplest application of the Fission GitHub action to deploy functions. You can use Fission and other GitHub actions to build powerful workflows and do a lot more. Try out the Fission GitHub action and reach out to us up on [Slack](http://slack.fission.io/) if you have any queries or interesting ideas that we can explore together.


--- 


**_Author:_**

* [Vishal Biyani](https://twitter.com/vishal_biyani)  **|**  [Fission Contributor](https://github.com/vishal-biyani)  **|**  CTO - [InfraCloud Technologies](http://infracloud.io/)