# Design Failure Strategies using Tekton

A CI/CD workflow can be represented in Tekton using Tekton `Pipeline`. Tekton `Pipeline` is a collection of Tekton `Tasks` which are represented in directed acyclic graph (DAG).

In general, a DAG includes more than one root nodes, more than one leaf nodes, a node can have multiple parent nodes (dependencies), and a node can be parent to multiple nodes. If we apply this to Tekton `Pipeline`, a `Pipeline` can be instantiated and terminated with multiple `Tasks` running in parallel. A `Pipeline` can have a `Task` which depends on multiple `Tasks`. There is an endless possibility of how one can arrange the list of `tasks`.

TODO: Create DAG of tasks

The DAG of `tasks` is built based on certain keywords in the `Pipeline`: 

- `runAfter`: A `task` can declare dependency on one or more `tasks` using `runAfter`. The `task` is only executed after all the `tasks` specified in `runAfter` (i.e. all parent `tasks`) exit successfully. For example, execute `deploy` task after all parent tasks (`unit-test`, `integration-test`, and `perf-test`) are successful. `deploy` task is skipped if any one of the parent tasks results in failure. 

```yaml
 tasks:
    - name: deploy
      runAfter: [ "unit-test", "integration-test", "perf-test" ]
```
   
-  `from`:  clause specified in `PipelineResources` also creates an edge between two tasks 

- task result specified in the `param` 

- finally: 

- when expressions: 



