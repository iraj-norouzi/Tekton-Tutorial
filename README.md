# Tekton-Tutorial
---------------------------------------------------------------------------------------------------------------------------
Advantages of Tekton
---------------------------------------------------------------------------------------------------------------------------
Talking about the architect Tekton
---------------------------------------------------------------------------------------------------------------------------
Prepair the environment
---------------------------------------------------------------------------------------------------------------------------
Install Kind
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
---------------------------------------------------------------------------------------------------------------------------
Implement the Kubernetes Cluster
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  apiServerAddress: "0.0.0.0"
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
  - containerPort: 6443
    hostPort: 6443
    protocol: TCP

EOF
---------------------------------------------------------------------------------------------------------------------------
Implement Ingress nginx 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
---------------------------------------------------------------------------------------------------------------------------
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
---------------------------------------------------------------------------------------------------------------------------
Apply sample application configuration
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/usage.yaml
---------------------------------------------------------------------------------------------------------------------------
Test ingress configuration
# should output "foo-app"
curl localhost/foo/hostname
# should output "bar-app"
curl localhost/bar/hostname
---------------------------------------------------------------------------------------------------------------------------
Tekton CLI
As the name implies, the Tekton CLI is the CLI tool that is used to manage Tekton
pipelines. This tool makes it easier to interact with Tekton components and is better than
relying on kubectl. It is available as a binary for all the major operating systems and can
be installed directly from the source code.
You can find the source code for this tool, as well as all its releases, in the Tekton GitHub
repository at https://github.com/tektoncd/cli.Introducing Tekton
25
This tool's installation instructions and usage will be covered in the next chapter of this
book.
---------------------------------------------------------------------------------------------------------------------------
Tekton Triggers
Tekton Triggers appeared as a child project of Tekton. It allows users to add a way to
launch pipelines based on webhooks automatically. A typical use case for this would be
to add your Tekton Trigger URL to your GitHub repository. This URL would be reached
each time an event occurs. Events would include things such as code pushes.
Using Triggers, you can launch a pipeline each time a git push is done in your code
base. GitHub will then send a POST request to your cluster with a payload containing all
the information about the event, such as the repository's URL and the branch name. Based
on this payload, you can trigger various pipelines if needed. For example, you could start
the pipeline so that it deploys the application to production each time new code is merged
into the master branch.
You can find out more information about Tekton Triggers, along with its source code, on
GitHub at https://github.com/tektoncd/triggers.
Tekton Triggers will be covered in the third part of this book.
---------------------------------------------------------------------------------------------------------------------------
Tekton Catalog
One of the goals of Tekton is to make Tasks as reusable as possible, so you don't have to
reinvent the wheel each time you need to create a new pipeline. Tekton provides a set of
standardized tasks that you can use on any of your CI/CD pipelines to help you with this.
Using an example that we discussed previously, you would almost always perform a git
clone task at the beginning of your pipeline to fetch the source code from a repository.
A premade task, managed by the Tekton team, can be installed directly into your cluster
so that you can use this task out of the box. These tasks have undergone much testing and
will allow you to reuse them in all your projects.
You can find the code for the actual Catalog at https://github.com/tektoncd/
hub, while all the available tasks are available in their own repository at https://
github.com/tektoncd/catalog.
Some of these tasks will be used in the last part of this book, when we start building real-
world examples of pipelines.26
A Cloud-Native Approach to CI/CD
---------------------------------------------------------------------------------------------------------------------------
Tekton Dashboard
The newest addition to the Tekton project is its Dashboard. Once it's been set up in
your cluster, it will provide you with a web-based UI so that you can view and manage
your Tekton components. It is built with React, and it can be installed directly into your
Kubernetes cluster. You can find the source code for this project on GitHub at https://
github.com/tektoncd/dashboard.
An overview of the Dashboard will be introduced in Chapter 3, Installation and Getting
Started.
These different projects might seem like a lot of content to cover, but we will slowly
introduce each of these new tools throughout this book. Before we start working with
these side projects, we need to understand what a Tekton Pipeline is.
---------------------------------------------------------------------------------------------------------------------------
Steps
Steps are the most basic units that you can use to create your pipeline. They represent a
single operation that is part of your greater CI/CD process. Examples of a step could be
running a test suite or compiling an application.
---------------------------------------------------------------------------------------------------------------------------
Tasks
A task is a collection of steps that will run in sequence. The steps inside a single task
should be related to each other. They typically represent a single operation that has to
be performed on the inputs. A more complex task could have multiple steps to prepare
the environment for the larger task. An example would be a test suite that needs some
dependencies before running. Installing these dependencies and running the test suites
would be two distinct steps in a single task.
The task will run in a single pod, which enables your steps to share a common volume and
some resources.
You can configure tasks in many ways. The most common configuration is to add multiple
parameters. Adding those parameters to your tasks will let you reuse the same task in a
different context.
You can find some pre-written tasks in the Tekton Catalog. Tasks for everyday operations
such as git clone or s2i are available and supported by the Tekton team. Those tasks
are good examples of more complex processes that have many parameters for maximum
reusability in all your pipelines.
While you can use tasks by themselves, they are usually used to build larger pipelines.Exploring Tekton's building blocks
29
---------------------------------------------------------------------------------------------------------------------------
Pipelines
A pipeline is a collection of tasks that will perform a series of operations on an input and
potentially provide you with some output. Pipelines describe your CI/CD workflow. A
typical pipeline would clone your code, run your tests, build an image, and deploy your
application into a cluster.
A pipeline will create several Kubernetes pods and run them in an order specified by the
user. They could be executed simultaneously for a faster output or sequentially if the task's
output is required for the next one to be completed.
Like tasks, you can parameterize pipelines so that you can reuse them in different
contexts. For example, a pipeline that takes a code base as input and pushes the resulting
image to a cluster could be used in various projects, if the code repository and image
name have been parameterized.
If a task is unsuccessful, the pipeline could be halted. This early stop is handy if you have a
task that runs a test suite. It could prevent pushing a bugged image into production.
---------------------------------------------------------------------------------------------------------------------------
Understanding TaskRuns and PipelineRuns
TaskRuns and PipelineRuns represent how their counterparts are executed; that is, the
task and the pipeline. They hold the current status of the task's and pipeline's execution
and provide you with more information on their performance, as shown in the following
diagram:
---------------------------------------------------------------------------------------------------------------------------
Installing the extensions
To install extensions, you can open the Extensions panel in VS Code with Ctrl + Shift + X.
From here, you can search for and install the following extensions:
• Kubernetes (ms-kubernetes-tools.vscode-kubernetes-tools) by
Microsoft
• YAML (redhat.vscode-yaml) by Red Hat
• Tekton Pipelines (redhat.vscode-tekton-pipelines) by Red Hat
To install the extensions, click on the Install button, as shown in the following screenshot:
---------------------------------------------------------------------------------------------------------------------------

Tekton pipelines:
Tekton pipelines allow us to create Pipelines that include some arbitrary tasks to be run.
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
---------------------------------------------------------------------------------------------------------------------------
Tekton triggers:

Tekton triggers help us to create endpoints that accept requests from git repositories such as push events that will be raised when a developer pushes their codes. 
kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml
---------------------------------------------------------------------------------------------------------------------------
Tekton Dashboards:
curl -sL https://raw.githubusercontent.com/tektoncd/dashboard/main/scripts/release-installer | \
  bash -s -- install latest --read-write

#kubectl apply --filename https://github.com/tektoncd/dashboard/releases/latest/download/tekton-dashboard-release.yaml
---------------------------------------------------------------------------------------------------------------------------
Implement Ingress for Tekton dashboard
# replace DASHBOARD_URL with the hostname you want for your dashboard
# the hostname should be setup to point to your ingress controller
DASHBOARD_URL=dashboard.domain.tld
kubectl apply -n tekton-pipelines -f - <<EOF
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tekton-dashboard
  namespace: tekton-pipelines
spec:
  rules:
  - host: $DASHBOARD_URL
    http:
      paths:
      - pathType: ImplementationSpecific
        backend:
          service:
            name: tekton-dashboard
            port:
              number: 9097
EOF
---------------------------------------------------------------------------------------------------------------------------
we will Run first task
tkn task start test-task  --showlog --no-color
---------------------------------------------------------------------------------------------------------------------------
Adding task parameters
One of the goals that you should aim for when building your tasks is to make them as
reusable as possible. A simple way to reuse a task in different contexts is to add parameters
to them. You can then substitute the values of those parameters in the steps that compose
your task.
tkn task start hello-param --showlog -p who=Joel

---------------------------------------------------------------------------------------------------------------------------
Adding a default value
---------------------------------------------------------------------------------------------------------------------------
