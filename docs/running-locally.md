# Running Locally

## Requirements

* [Go 1.18](https://golang.org/dl/)
* [Yarn](https://classic.yarnpkg.com/en/docs/install/#mac-stable)
* [Docker](https://docs.docker.com/get-docker/)
* [protoc](http://google.github.io/proto-lens/installing-protoc.html)
* [jq](https://stedolan.github.io/jq/download/)
* A local Kubernetes cluster (`k3d`, `kind`, or `minikube`)

We recommend using [K3D](https://k3d.io/) to set up the local Kubernetes cluster since this will allow you to test RBAC
set-up and is fast. You can set-up K3D to be part of your default kube config as follows:

```shell
k3d cluster start --wait
```

Alternatively, you can use [Minikube](https://github.com/kubernetes/minikube) to set up the local Kubernetes cluster.
Once a local Kubernetes cluster has started via `minikube start`, your kube config will use Minikube's context
automatically.

⚠️ Do not use Docker for Desktop, it does not support Kubernetes RBAC (i.e. `kubectl auth can-i` always
returns `allowed`).

## Developing locally

Close the Git repo into: `$(GOPATH)/src/github.com/argoproj/argo-workflows`. Any other path will mean the code
generation does not work.

Add the following to your `/etc/hosts`:

```
127.0.0.1 dex
127.0.0.1 minio
127.0.0.1 postgres
127.0.0.1 mysql
```

To start:

* The controller, so you can run workflows.
* MinIO (http://localhost:9000, use admin/password) so you can use artifacts:

Run:

```shell
make start 
```

Make sure you don't see any errors in your terminal.

You can submit a workflow for testing using `kubectl`:

```shell
kubectl create -f examples/hello-world.yaml 
```

If you made changes to how the executor, you need to build the image:

```shell
make argoexec-image
```

To also start the API on https://localhost:2746:

```shell
make start API=true
```

To also start the UI on http://localhost:8080 (`UI=true` implies `API=true`):

```shell
make start UI=true
```

If you are making change to the CLI, you can build it:

```shell
make cli 
./dist/argo submit examples/hello-world.yaml ;# new CLI is created as `./dist/argo` 
```

To test the workflow archive, use `PROFILE=mysql` or `PROFILE=postgres`:

```shell
make start PROFILE=mysql
```

You'll have, either:

* Postgres on http://localhost:5432, run `make postgres-cli` to access.
* MySQL on http://localhost:3306, run `make mysql-cli` to access.

To test SSO integration, use `PROFILE=sso`:

```shell
make start UI=true PROFILE=sso
```

### Running E2E tests locally

Start up the Argo Workflows using the following:

```shell
make start PROFILE=mysql AUTH_MODE=client STATIC_FILES=false API=true 
```

#### Running One Test

In most cases, you want to run the test that relates to your changes locally. You should not run all the tests suites.
Our CI will run those concurrently when you create a PR, which will give you feedback much faster.

Find the test that you want to run in `test/e2e`

```shell
make TestArtifactServer'  
```

#### Running A Set Of Tests

You can find the build tag at the top of the test file.

  ```go
//go:build api
```

You need to run `make test-{buildTag}`, so for `api` that would be:

```shell
make test-api
```

#### Diagnosing Test Failure

Tests often fail, that's good. To diagnose failure:

* Run `kubectl get pods`, are pods in the state you expect?
* Run `kubectl get wf`, is your workflow in the state you expect?
* What do the pod logs say? I.e. `kubectl logs`.
* Check the controller and argo-server logs. These are printed to the console you ran `make start` in. Is anything
  logged at `level=error`?

If tests run slowly or time out, factory reset your Kubernetes cluster.

## Committing

Before you commit code and raise a PR, always run:

```shell
make pre-commit -B
```

Please do the following when creating your PR:

* Sign-off your commits.
* Use [Conventional Commit messages](https://www.conventionalcommits.org/en/v1.0.0/).
* Suffix the issue number.

Examples:

```shell
git commit --signoff -m 'fix: Fixed broken thing. Fixes #1234'
```

```shell
git commit --signoff -m 'feat: Added a new feature. Fixes #1234'
```

## Troubleshooting

* When running `make pre-commit -B`, if you encounter errors like
  `make: *** [pkg/apiclient/clusterworkflowtemplate/cluster-workflow-template.swagger.json] Error 1`, ensure that you
  have checked out your code into `$(GOPATH)/src/github.com/argoproj/argo-workflows`.
* If you encounter "out of heap" issues when building UI through Docker, please validate resources allocated to Docker.
  Compilation may fail if allocated RAM is less than 4Gi.

## Using Multiple Terminals

I run the controller in one terminal, and the UI in another. I like the UI: it is much faster to debug workflows than
the terminal. This allows you to make changes to the controller and re-start it, without restarting the UI (which I
think takes too long to start-up).

As a convenience, `CTRL=false` implies `UI=true`, so just run:

```shell
make start CTRL=false
```
