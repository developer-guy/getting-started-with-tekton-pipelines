# What is Tekton?
Tekton is a cloud-native solution for building CI/CD pipelines. It consists of Tekton Pipelines, which provides the building blocks, and of supporting components, such as Tekton CLI and Tekton Catalog, that make Tekton a complete ecosystem. Tekton is part of the CD Foundation, a Linux Foundation project.

Tekton installs and runs as an extension on a Kubernetes cluster and comprises a set of Kubernetes Custom Resources that define the building blocks you can create and reuse for your pipelines. Once installed, Tekton Pipelines becomes available via the Kubernetes CLI (kubectl) and via API calls, just like pods and other resources.

> Credit: https://tekton.dev/docs/overview/

# Tekton Core Components
Tekton introduces the concept of Tasks, which specify the workloads you want to run:

* Task - defines a series of ordered Steps, and each Step invokes a specific build tool on a specific set of inputs and produces a specific set of outputs, which can be used as inputs in the next Step.

* Pipeline - defines a series of ordered Tasks, and just like Steps in a Task, a Task in a Pipeline can use the output of a previously executed Task as its input.

* TaskRun - instantiates a specific Task to execute on a particular set of inputs and produce a particular set of outputs. In other words, the Task tells Tekton what to do, and a TaskRun tells Tekton what to do it on, as well as any additional details on how to exactly do it, such as build flags.

* PipelineRun - instantiates a specific Pipeline to execute on a particular set of inputs and produce a particular set of outputs to particular destinations.

Each Task executes in its own Kubernetes Pod. Thus, by default, Tasks within a Pipeline do not share data. To share data among Tasks, you must explicitly configure each Task to make its outputs available to the next Task and to ingest the outputs of a previously executed Task as its inputs, whichever is applicable.

