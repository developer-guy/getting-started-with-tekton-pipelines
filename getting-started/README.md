# Prerequisites
> NOTE: I did this demo on macOS 10.15.7 Catalina, so if you are in the same environment you can use "brew" a package manager for macOS to do the installements.
* kind v0.10.0
* tkn 0.16.0
* kubectl v1.20.2

# Hands On
The first things first, we need to install Tekton to our local Kubernetes cluster. In order to do that we need to provision our local Kubernetes cluster, so we'll use ["kind"]() (Kubernetes in Docker).

Lets start our local Kubernetes cluster.
```bash
$ kind create cluster
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.20.2) 🖼
 ✓ Preparing nodes 📦
 ✓ Writing configuration 📜
 ✓ Starting control-plane 🕹️
 ✓ Installing CNI 🔌
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a nice day! 👋
```

Then, we need to install Tekton itself. There are various ways to do that, first, we can apply a plain YAML manifest file which is [install.yaml](../install.yaml) in this case, another way of installing Tekton is using it's operator, to get more information you can take a look at [here.](https://github.com/tektoncd/operator).

Lets install Tekton using plain YAML method.
```bash
$ kubectl apply -f ../install.yaml
Found existing alias for "kubectl apply -f". You should use: "kaf"
namespace/tekton-pipelines created
podsecuritypolicy.policy/tekton-pipelines created
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-controller-cluster-access created
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-controller-tenant-access created
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-webhook-cluster-access created
role.rbac.authorization.k8s.io/tekton-pipelines-controller created
role.rbac.authorization.k8s.io/tekton-pipelines-webhook created
role.rbac.authorization.k8s.io/tekton-pipelines-leader-election created
serviceaccount/tekton-pipelines-controller created
serviceaccount/tekton-pipelines-webhook created
...
```

Lets verify the installation.
```bash
$ kubectl get pods --namespace tekton-pipelines
Found existing alias for "kubectl get pods". You should use: "kgp"
NAME                                           READY   STATUS    RESTARTS   AGE
tekton-pipelines-controller-769bcc6996-dvzm9   1/1     Running   0          60s
tekton-pipelines-webhook-76d9b48fff-mqjjx      1/1     Running   0          60s
```

Now, we are ready to go to create our first task using Task custom resource. If you haven't read the descriptions of the core components of Tekton from this [README.md](../README.md) yet, please read it from here first.

Here is the definition of it.
```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello
spec:
  steps:
    - name: hello
      image: ubuntu
      command:
        - echo
      args:
        - "Hello World!"
```

Lets create it using "kubectl".
```bash
$ kubectl apply -f task-hello.yaml
task.tekton.dev/hello created
```

Lets verify it in both ways using kubectl and tkn cli.
```bash
$ kubectl get tasks.tekton.dev
NAME    AGE
hello   89s
```

```bash
$ tkn task list
NAME    DESCRIPTION   AGE
hello                 2 minutes ago
```

Lets create another Task because we'll combine them together within the Pipeline resource, we should do the same steps above for our "goodbye" task.
```bash
$ kubectl apply -f task-goodbye.yaml
task.tekton.dev/goodbye created
```

Lets verify it.
```bash
$ tkn task list
NAME    DESCRIPTION   AGE
hello                 2 minutes ago
goodbye               1 minutes ago
```

Lets combine them together within the Pipeline resource.
```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: hello-goodbye
spec:
  tasks:
  - name: hello
    taskRef:
      name: hello
  - name: goodbye
    runAfter:
     - hello
    taskRef:
      name: goodbye
```

You should notice that we ordered our Tasks, so first hello task will kick off before the goodbye task, once its finished, goodbyte task will start to do its job.

Lets create this.
```bash
$ kubectl apply -f hello-goodbye.yaml
pipeline.tekton.dev/hello-goodbye created
```

In order to run this Pipeline we need PipelineRun resource, think of it a runtime for the Pipeline, lets run it using tkn cli, btw same concern is exists for the Task one, in order to run Task you need to create TaskRun resource, it is simple just run "tkn task start hello", it will create a TaskRun and apply it to the cluster.
```bash
$ tkn pipeline start hello-goodbye
 tkn pipeline start hello-goodbye
PipelineRun started: hello-goodbye-run-k4q48

In order to track the PipelineRun progress run:
tkn pipelinerun logs hello-goodbye-run-k4q48 -f -n default
```

You should notice that, once we start this pipeline, a pod is started to do its job, lets take a look the pipeline logs.
```bash
$ tkn pipelinerun logs hello-goodbye-run-k4q48 --last -f
[hello : hello] Hello World!
[goodbye : goodbye] Goodbye World!
```

Great, you created and run your first Pipeline using Tekton.
