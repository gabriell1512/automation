# Kubernetes Automation -- It starts to be fun when you can automate everything!

## Introduction

Last semester, we worked with GitHub to keep a version control history of our code. For this part of the curriculum, we can try to use some other cool things regarding GitHub that we haven't tackled before.

GitHub contains a feature called GitHub actions. This is very useful to set up automation pipelines that get triggered either manually or automatically using a set of triggers.
As of a few years, it's also possible to run on-premises runners that execute commands in the order you need.

In this assignment, we will explore the integration of Kubernetes with GitHub Actions, to set up a powerful and versatile continuous integration and continuous deployment (CI/CD) pipeline. By employing custom GitHub Actions runners, we will learn how to optimize and tailor our workflows to suit our specific project requirements.

# Getting to know GitHub Actions

We will get started with a Basic GitHub Actions workflow now, so you know how everything works for the other coming courses.

### Create a new repository

We will need to create a new GitHub repository first, so we can add the GitHub Actions Runner to the repo.
- [ ] Create a new GitHub repository using your account
- [ ] Clone it to your PC
- [ ] Create a new branch on your repository called `01_DockerTest`


### Basic Docker application

GitHub Actions work best when a set of repeated actions have to be executed. This is why we will test it using a simple Dockerfile to create a very basic HTML website at first.



- Create a Dockerfile
    ```dockerfile
    FROM nginx:alpine
    COPY . /usr/share/nginx/html
    ```
- Create a basic HTML page `index.html`
    ```html
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Hello world</title>
    </head>
    <body>
        <h1>Hello world!</h1>
    </body>
    </html>
    ```

:::info
You can test this image by building it (I tagged it as `hello-world-nginx:v1`).
Then you run it, with a port mapping to `chosen_port:80`. If you use VSCode, you can then `Open Browser` on the container, and it will tunnel the chosen port, and open it in the browser. You will see your **Hello world** content.
:::

- [ ] Push all of this content to a new branch called `01_DockerTest`. Your main branch stays clean and empty (apart from a Readme, if you want).

## Creating a GitHub Actions flow

- [ ] Go to the Actions tab on your GitHub Repository.
- [ ] Select the `Simple workflow`
    ![Simple Workflow Action by GitHub](https://i.imgur.com/LWYtaCy.png)
- [ ] Change the name of the Workflow to `My First Workflow`
- [ ] Commit this file to the Main branch
- [ ] Take a look at the output of your Job

:::success
**QUESTION**
What did the `multi-line script` return ?
```text
Run echo Add other actions to build,
  echo Add other actions to build,
  echo test, and deploy your project.
  shell: /usr/bin/bash -e {0}
Add other actions to build,
test, and deploy your project.
```
:::

### Changing the workflow file from your IDE

Open the repository that you cloned into your PC and 
The file was added under `.github/workflows/blank.yml`

- [ ] Adapt the Workflow so you can build your Dockerfile, with the `hello-world-nginx:v1` flag. Try to make the Actions **start and run** the Docker container, map the right port using the right flags. You should be able to do that using simple Actions in GitHub.

:::success
**QUESTION**
What did you change ? Show the changes from your workflow file and explain it a little.
```yaml
name: CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Build and push Docker image
        uses: docker/build-push-action@v2
        with:
          context: .
          push: false
          tags: hello-world-nginx:v1

      - name: Run Docker image
        uses: docker/run-action@v1
        with:
          image: hello-world-nginx:v1
          options: -p 80:80

    i added a build and run for the docker image
```
:::

- [ ] Push your changes to the main branch. 

:::success
**QUESTION**
Did your workfow fail? If so, why did it?
```text
// Answer here
```
:::

:::spoiler Click here for the coming instructions, as they give you the answer to the previous question!
- [ ] Create a pull request from the `01_DockerTest` branch, into your `main`-branch.
- [ ] Your pipeline will start already, as it also triggers on `pull_requests`
:::
- [ ] If everything went well, you got the content of your nginx application inside of the GitHub Actions flow.
- [ ] You can now merge your Pull Request into your Main branch.

:::info
Try to experiment with the different options of the Docker build actions. There are a few community-available tasks you can experiment with.
This is one I like to use a lot:
https://github.com/marketplace/actions/build-and-push-docker-images

Using GitHub Actions, you can automate the tags to include the `branch` and the `commit-sha`.
Try that as well.
:::

Now, this was just a demo. Of course the tasks we will do will be a little bit more complex.

As long as you already get the concepts, I am happy with that.


### Running the runner on your laptop

Apart from using the GitHub Actions Runners that are hosted by GitHub themselves, we can get our own local runners as well. This is useful if you want to get more build time, or you require some bigger machines to perform your actions. We also need this if we have to access some things on our own network, such as the Kubernetes cluster that you're running on your own laptop.

Documentation about GitHub Actions Runners can be found [here](https://docs.github.com/en/actions/hosting-your-own-runners/adding-self-hosted-runners)

Follow these instructions to get started. **Configure the runner for your OS**.

:::spoiler Linux and MacOS

#### Downloading the Runner

```shell=
student@segers-nathan:~$ mkdir actions-runner && cd actions-runner
student@segers-nathan:~/actions-runner$ curl -o actions-runner-linux-x64-2.283.3.tar.gz -L https://github.com/actions/runner/releases/download/v2.283.3/actions-runner-linux-x64-2.283.3.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   647  100   647    0     0   6103      0 --:--:-- --:--:-- --:--:--  6103
100 70.5M  100 70.5M    0     0  8372k      0  0:00:08  0:00:08 --:--:-- 10.3M
student@segers-nathan:~/actions-runner$ echo "09aa49b96a8cbe75878dfcdc4f6d313e430d9f92b1f4625116b117a21caaba89  actions-runner-linux-x64-2.283.3.tar.gz" | shasum -a 256 -c
actions-runner-linux-x64-2.283.3.tar.gz: OK
student@segers-nathan:~/actions-runner$ tar xzf ./actions-runner-linux-x64-2.283.3.tar.gz
```

#### Configuring the Runner
```shell=
student@segers-nathan:~/actions-runner$ ./config.sh --url <repo> --token <token-redacted>

--------------------------------------------------------------------------------
|        ____ _ _   _   _       _          _        _   _                      |
|       / ___(_) |_| | | |_   _| |__      / \   ___| |_(_) ___  _ __  ___      |
|      | |  _| | __| |_| | | | | '_ \    / _ \ / __| __| |/ _ \| '_ \/ __|     |
|      | |_| | | |_|  _  | |_| | |_) |  / ___ \ (__| |_| | (_) | | | \__ \     |
|       \____|_|\__|_| |_|\__,_|_.__/  /_/   \_\___|\__|_|\___/|_| |_|___/     |
|                                                                              |
|                       Self-hosted runner registration                        |
|                                                                              |
--------------------------------------------------------------------------------

# Authentication


√ Connected to GitHub

# Runner Registration

Enter the name of the runner group to add this runner to: [press Enter for Default]

Enter the name of runner: [press Enter for segers-nathan]

This runner will have the following labels: 'self-hosted', 'Linux', 'X64'
Enter any additional labels (ex. label-1,label-2): [press Enter to skip]

√ Runner successfully added
√ Runner connection is good

# Runner settings

Enter name of work folder: [press Enter for _work]

√ Settings Saved.

student@segers-nathan:~/actions-runner$ ./run.sh
```

:::

### Using the self-hosted runner

- [ ] Adapt your Workflow to work on your self-hosted runner now.
    ```yaml
    # Use this YAML in your workflow file for each job
    runs-on: self-hosted
    ```
    
Try if everything still runs well.


# Kubernetes Application Updates

To update an application in Kubernetes, all we need to do is make sure we have a new version of the Docker image ready and pushed.
In our case, most of our Docker images reside on the GitHub Container Registry, which is a private registry we can use. In that case, it has to be prefixed with `ghcr.io` in the tag. As that one is a private registry, we also need to log in to the registry before we can use it.

In Kubernetes, we can do so using a `PullSecret` which you can use.

If you prefer to work with Docker Hub and public repositories for the sake of the exercise, you're very welcome to do so.

Once you have a new version of your application, you might have tagged is as `ghcr.io/nathansegers/my-app:2.0.0`.
:::info
A nice trick is to always include a few tags, such as `2.0.0` but also `main`, `dev`, `production` and the `git-sha` tag, which is the ID of the Git commit you are referring to.
:::

Simply updating your Kubernetes YAML file should be more than enough (Unless you have a few more code changes that drastically changed the structure of your app: Database connections or even a different port it's listening on ...)

Read on to find out how to automate these changes in a GitHub Actions pipeline.

## How applications update in Kubernetes

By default, Kubernetes has a Rolling Update strategy, which means that everytime a new version is being rolled out, it will gradually update.
Depending on the settings you choose, a few extra pods will be added (the `MaxSurge` option) which will already contain the new version of the application.
While that update is happening, your older versions of the applications need to be destroyed. This is called the `MaxUnavailable` option. While it's adding new pods, it's already removing some of the older ones.

In the theory class I exlained the difference between a Blue-green deployment and a Canary Deployment, which are two methods of redeploying an application into production to gradually roll it out.

You can achieve these things in Kubernetes by using a Load Balancer such as Ingress, they can Split the traffic accordingly.

For the MLOps course, it's fine to know this theoretically, and you shouldn't experiment with it in a practical scenario. If you want to do so, feel free!


Now that we know that there is an easy option to roll out a new version in Kubernetes, we can try to automate it by setting up a GitHub Action to do all the repetitive steps we used to do manually.


## GitHub Actions + Kubectl

### First trials

GitHub Actions has a way of connecting to your local Kubernetes instance by using Kubectl, the same interface you are using yourself and using the self-hosted GitHub Actions runner.

To test out if your local runner can access your Kubernetes cluster, you can try to install a Kubernetes object with the Kubectl commands.

1. Create a new workflow
2. Trigger it manually `workflow_dispatch` or through a `push` on the main-branch
3. Make sure it uses the `self-hosted` runner this time.
4. Add the right actions:
    - The action to pull your source code
    - A KubeCTL action which will apply a Kubernetes object to your cluster
    - A KubeCTL action to check for the pods in your namespace
5. Find a way to generate a random name for the Kubernetes Pod object.
You can find an example file in here:

```yaml=
apiVersion: v1
kind: Pod
metadata:
  name: mypod-{ID} # Try to make this dynamically generated (random)
  namespace: <your-namespace> # Enter your namespace here
  labels:
    name: mypod
spec:
  containers:
  - name: mypod
    image: rancher/hello-world
```

:::success
**QUESTION**
How did you make the name randomised?
```text
// Answer here
```
:::

### Getting Helm involved

As we know, Helm is great to do automations of the Kubernetes installations. We simply need to enter a command like `helm upgrade --install ...` with all the necessary parameters to install a chart into our cluster.

The useful thing to combine with GitHub Actions is that you can override the `values.yaml` file from the command line as well.

You can even use a special Action / Task that is specialised in editting YAML files. That way you can easily specify a new version of the Docker image.

Try to install a Helm chart into your Kubernetes cluster, with the right GitHub Actions that you can confirm. You can use the same logic as you used above for the Kubernetes part.

:::success
**QUESTION**
How does the command for a Helm install look like with the flags added ?
```text
// Answer here
```
:::



## Complete automation

This quick checklist / manual can help you in figuring out a good structure for the GitHub Actions.

1. Clone the GitHub repository
2. Optional: Perform some automated checks on the code if needed (Unit Test, Integration Test, Cyclomatic Complexity, Code Coverage ...)
3. Build the Docker Image with a new tag (`:latest`, `:<GIT-SHA>`). We like to refer to the GIT-SHA as it's always unique and refers back to the Git commit we performed.
4. Push the Docker Image (to GHCR or Docker Hub).
5. Deploy using plain Kubernetes (without Helm) or with Helm

**Kubernetes without Helm**

1. Create a Kubernetes namespace, if needed
2. Change the Kubernetes YAML files
3. Apply the Kubernetes YAML files
4. Test the final result

**Kubernetes with Helm**
1. Create a Kubernetes namespace, if needed
2. Override the `values.yaml` file using a command line interface flag or with a specialised task.
3. Upgrade (or install) the Helm chart
4. Test the final result

### A few notes

A few things to note using the automation with GitHub Actions:
- Make sure it works when you have just installed a clean Kubernetes cluster: Cold start
    - [ ] Create namespace **If doesn't exist**
    - [ ] Change properties if needed, or use default properties
    - [ ] Update **OR install**. Don't forget that your `:latest` tag will not update the Kubernetes cluster, as it will only have an effect if the Kubernetes' **Desired State** is different than it was before. That's why we prefer to use **Semantic Versioning** for our Docker Image tags. If you want to overcome that issue, you can use the **`ImagePullPolicy: Always`** property in your Kubernetes deployment container specs.
- Don't have any secrets into your GitHub repository
- See if your GitHub Actions runner is active
- Try automated tests in your project


{%hackmd jnrJ5Un2RkmKumwxf1ml5Q %}

###### tags: `MLOps@Home` `Kubernetes` `MCT` `2022-2023` `Syllabus`