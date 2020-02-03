# actions-runner-controller

This controller operates self-hosted runners for GitHub Actions on your Kubernetes cluster.

## Motivation

[GitHub Actions](https://github.com/features/actions) is very useful as a tool for automating development. GitHub Actions job is run in the cloud by default, but you may want to run your jobs in your environment. [Self-hosted runner](https://github.com/actions/runner) can be used for such use cases, but requires the provision of a virtual machine instance and configuration. If you already have a Kubernetes cluster, you'll want to run the self-hosted runner on top of it.

*actions-runner-controller* makes that possible. Just create a *Runner* resource on your Kubernetes, and it will run and operate the self-hosted runner of the specified repository. Combined with Kubernetes RBAC, you can also build simple Self-hosted runners as a Service.

## Installation

First, install *actions-runner-controller* with a manifest file. This will create a *actions-runner-system* namespace in your Kubernetes and deploy the required resources.

```
$ kubectl -f https://github.com/summerwind/actions-runner-controller/releases/download/latest/actions-runner-controller.yaml
```

Set your access token of GitHub to the secret. `${GITHUB_TOKEN}` is the value you must replace with your access token. This token is used to register Self-hosted runner by *actions-runner-controller*.

```
$ kubectl create secret generic controller-manager --from-literal=github_token=${GITHUB_TOKEN} -n actions-runner-system
```

## Usage

To launch Self-hosted runner, you need to create a manifest file includes *Runner* resource as follows. This example launches a self-hosted runner with name *example-runner* for the *summerwind/actions-runner-controller* repository.

```
$ vim runner.yaml
```
```
apiVersion: actions.summerwind.dev/v1alpha1
kind: Runner
metadata:
  name: example-runner
spec:
  repository: summerwind/actions-runner-controller
```

Apply the created manifest file to your Kubernetes.

```
$ kubectl apply -f runner.yaml
```

You can see that the Runner resource has been created.

```
$ kubectl get runners
NAME             AGE
example-runner   1m
```

You can also see that the runner pod has been running.

```
$ kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
example-runner 2/2     Running   0          1m
```

The runner you created has been registerd to your repository.

<img width="756" alt="Actions tab in your repository settings" src="https://user-images.githubusercontent.com/230145/73618667-8cbf9700-466c-11ea-80b6-c67e6d3f70e7.png">

Now your can use your self-hosted runner. See the [documentation](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/using-self-hosted-runners-in-a-workflow) on how to run a job with it.
