# Goodbye Tekton Condition CRD

Pipeline is one of the core building blocks in designing a CI/CD use case on Kubernetes using Tekton. Tekton Pipeline is a collection of `Tasks` which are executed based on how they are arranged. These tasks can be represented as a graph in which each node represents a `Task` and can be arranged in many different ways:

* All tasks(`task-A`, `task-B`, and `task-C`) running in parallel. All three tasks starts executing independently and finishes execution irrespective of success/failure of other tasks. This kind of arrangement applies to a CI/CD use case such as running unit tests and integration tests in parallel. 

```yaml
  tasks:
    - name: task-A
      taskRef:
        name: task1
    - name: task-B
      taskRef:
        name: task2
    - name: task-C
      taskRef:
        name: task3
``` 

* Design sequence of tasks running one after the other using `runAfter`. Let's look at a realistic example. PipelineRun instantiates `build-image` and waits for it to finish. If `build-image` exits with success, PipelineRun instantiates `e2e-tests` otherwise PipelineRun exits with failure without executing rest of the tasks. Same process applies to the next task. This is the ideal arrangement but hardly works in real world. It's very common to run into task failures and require further processing in case of such failures.

```yaml
  tasks:
    - name: build-image
      taskRef:
        name: build-image
    - name: e2e-tests
      runAfter: [ build-image ]
      taskRef:
        name: e2e-tests
    - name: deploy
      runAfter: [ e2e-tests ]
      taskRef:
        name: deploy
``` 

* Design failure strategies using `finally`. `PipelineRun` executes tasks specified under `finally` section after all tasks under `tasks` section are completed regardless success or failure i.e. `finally` tasks start executing after all tasks are completed successfully or one of the tasks exit with failure. `finally` tasks are all executed in parallel just before exiting the `PipelineRun`. `PipelineRun` exit status contains `TaskRuns` for all tasks including `finally` tasks.

```yaml
  tasks:
    - name: build-image
      taskRef:
        name: build-image
    - name: e2e-tests
      runAfter: [ build-image ]
      taskRef:
        name: e2e-tests
    - name: deploy
      runAfter: [ e2e-tests ]
      taskRef:
        name: deploy
  finally:
    - name: cleanup
      taskRef:
        name: cleanup
``` 

* So far looks great but what about running a check before executing a task? `PipelineRun` can be designed to include conditional tasks using `Condition` CRD to run a check before executing that task. `Condition` CRD, just like a `task`, spawns a new `pod` in a Kubernetes cluster to run that check. Depending on the outcome of that `Condition` CRD, a task is executed or declared `failure` with `conditionCheckFailed`. 

```yaml
apiVersion: tekton.dev/v1alpha1
kind: Condition
metadata:
  name: check-if-dev
spec:
  params:
    - name: "env"
  check:
    image: alpine
    script: |
      if [ $(params.env) != "dev" ]; then
        exit 1
      fi 
---

apiVersion: tekton.dev/v1alpha1
kind: Condition
metadata:
  name: check-if-stage
spec:
  params:
    - name: "env"
  check:
    image: alpine
    script: |
      if [ $(params.env) != "stage" ]; then
        exit 1
      fi 
---

apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: build-test-deploy-in-all-env
spec:
  params:
    - name: "env"
  tasks:
    ...
    - name: deploy-dev
      taskRef:
        name: deploy
      conditions:
        - conditionRef: "check-if-dev"
          params:
            - name: "env"
              value: "$(params.env)"
    - name: deploy-stage
      taskRef:
        name: deploy
      conditions:
        - conditionRef: "check-if-stage"
          params:
            - name: "env"
              value: "$(params.env)"
---

apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: build-test-deploy-in-dev
spec:
  pipelineRef:
    name: build-test-deploy-in-all-env
  params:
    - name: "env"
      value: "dev"
---
``` 

`Condition` CRD played an important factor in implementing conditional task or implementing guarded task but soon became too expensive for simple conditional check e.g. match the `pipeline` param and only execute if param matches a specific string like "dev", "stage", etc.

This sample `PipelineRun` results in a pod creation for each `conditionRef` i.e. (1) `build-test-deploy-in-dev-deploy-dev-zvw84-check-if-dev-0--flppx` and (2) `build-test-deploy-in-dev-deploy-stage-x5vbv-check-if-stag-xn6bz`:

```shell script
kubectl get pods
NAME                                                              READY   STATUS      RESTARTS   AGE
build-test-deploy-in-dev-deploy-dev-q797c-pod-n8nks               0/1     Completed   0          2m46s
build-test-deploy-in-dev-deploy-dev-zvw84-check-if-dev-0--flppx   0/1     Completed   0          2m55s
build-test-deploy-in-dev-deploy-stage-x5vbv-check-if-stag-xn6bz   0/1     Error       0          2m55s
```

The task for which the condition resulted in no match i.e. `deploy-stage` in this sample `Pipeline` results in failure:

```shell script
tkn pr describe build-test-deploy-in-dev
Name:              build-test-deploy-in-dev
Namespace:         default
Pipeline Ref:      build-test-deploy-in-all-env
Service Account:   default
Timeout:           1h0m0s
Labels:
 tekton.dev/pipeline=build-test-deploy-in-all-env

🌡️  Status

STARTED         DURATION     STATUS
2 minutes ago   14 seconds   Succeeded(Completed)

⚓ Params

 NAME    VALUE
 ∙ env   dev

🗂  Taskruns

 NAME                                            TASK NAME      STARTED         DURATION    STATUS
 ∙ build-test-deploy-in-dev-deploy-stage-75bmz   deploy-stage   ---             ---         Failed(ConditionCheckFailed)
 ∙ build-test-deploy-in-dev-deploy-dev-q797c     deploy-dev     2 minutes ago   5 seconds   Succeeded
```

## When Expressions

Tekton Pipeline has implemented a feature called When Expressions which is similar to Kubernetes' Match Expressions, in that it takes an input, operator and values:

```yaml
  tasks:
    - name: deploy-dev
      when:
        - input: "$(params.env)"
          operator: in
          values: [ "dev" ]
      taskRef:
        name: deploy-dev
```

- `Input` can be a static string or Pipeline `param` or task `results`.

- `Operator` represents an `Input`'s relationship to a set of `Values`. Two operators `in` and `notin` are supported with the initial implementation.

- `Values` is a list of string values. The list can contain static strings and/or Pipeline `params` and/or task `results`.

When expression simplifies the pipeline and builds efficient `PipelineRun` without creating unnecessary pods to match pipeline params which can be done by the `Pipeline` controller. Unlike `Conditions` CRD, the task with When Expressions failure are marked as `skipped` and results in `Successful` PipelineRun.

Let's rewrite the above example with `Conditions` CRD using When Expressions:

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: build-test-deploy-in-all-env
spec:
  params:
    - name: "env"
  tasks:
    - name: deploy-dev
      taskRef:
        name: deploy
      when:
        - input: "$(params.env)"
          operator: in
          values: [ "dev" ]
    - name: deploy-stage
      taskRef:
        name: deploy
      when:
        - input: "$(params.env)"
          operator: in
          values: [ "stage" ]
---

apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: build-test-deploy-in-dev
spec:
  pipelineRef:
    name: build-test-deploy-in-all-env
  params:
    - name: "env"
      value: "dev"
---
```

Notice, only one pod is created for the task which was executed:

```shell script
kubectl get pods
NAME                                                  READY   STATUS      RESTARTS   AGE
build-test-deploy-in-dev-deploy-dev-6h8vj-pod-9ffx2   0/1     Completed   0          16s
```

No misleading task failures under the list of `TaskRuns`:

```shell script
 tkn pr describe build-test-deploy-in-dev
Name:              build-test-deploy-in-dev
Namespace:         default
Pipeline Ref:      build-test-deploy-in-all-env
Service Account:   default
Timeout:           1h0m0s
Labels:
 tekton.dev/pipeline=build-test-deploy-in-all-env

🌡️  Status

STARTED          DURATION    STATUS
48 seconds ago   6 seconds   Succeeded(Completed)

⚓ Params

 NAME    VALUE
 ∙ env   dev

🗂  Taskruns

 NAME                                          TASK NAME    STARTED          DURATION    STATUS
 ∙ build-test-deploy-in-dev-deploy-dev-6h8vj   deploy-dev   48 seconds ago   6 seconds   Succeeded
```

Retrieve a list of `skipped` tasks:

```shell script
kubectl get pr build-test-deploy-in-dev -o json | jq .status.skippedTasks
[
  {
    "name": "deploy-stage"
  }
]
```

When expression can be used in many different use cases:

- Guard executing tasks based on the result produced by the parent tasks.
- Design generic pipelines and use it in different environments, regions, cloud providers, etc. For example, one single pipeline can be designed to deploy an application to `dev`, `qa`, `stage` and `prod`.
- Disable one single task or a list of tasks from the entire pipeline.
- Perform extra processing in terms of a task or a list of tasks depending on the input from `PipelineRun`.

To summarize, [Tekton Pipeline v0.16](https://github.com/tektoncd/pipeline/releases/tag/v0.16.3) includes When expressions which are concise, simple, and efficient. Please check [Pipeline repo](https://github.com/tektoncd/pipeline/blob/v0.16.3/docs/pipelines.md#guard-task-execution-using-whenexpressions) for detailed usage and more examples using When Expressions. 



