Cron Feed Trigger

# manifest.yaml

```yaml
package:
    name: helloworld
    actions:
        helloworld:
            location: src/helloworld.js
            runtime: nodejs:6
            inputs:
                name:
                    type: string
                    description: name of a person
                place:
                    type: string
                    description: location of a person
            outputs:
                payload:
                    type: string
                    description: a simple greeting message, Hello World!
    triggers:
        cron-trigger:
            source: /whisk.system/alarms/alarm
    rules:
        hellowroldOnCron:
            action: helloworld
            trigger: cron-trigger
```

# deployment.yaml

```yaml
application:
    name: SampleHelloWorld
    namespace: _
    package:
        name: helloworld
        actions:
            helloworld:
                inputs:
                    name: Amy
                    place: Paris
        triggers:
            cron-trigger:
                inputs:
                    cron: "0 */12 * * *"
```

# deployment 

```
 ~/IBM/GoWorkspace/src/github.com/openwhisk/openwhisk-wskdeploy/wskdeploy
2017/03/15 09:20:49 trigger feed source:/whisk.system/alarms/alarm
         ____      ___                   _    _ _     _     _
        /\   \    / _ \ _ __   ___ _ __ | |  | | |__ (_)___| | __
   /\  /__\   \  | | | | '_ \ / _ \ '_ \| |  | | '_ \| / __| |/ /
  /  \____ \  /  | |_| | |_) |  __/ | | | |/\| | | | | \__ \   <
  \   \  /  \/    \___/| .__/ \___|_| |_|__/\__|_| |_|_|___/_|\_\
   \___\/              |_|

Packages:
Name: helloworld
    bindings:
  * action: helloworld
    bindings:
        - name: name value: Amy
        - name: place value: Paris


Triggers:
* trigger: Every12Hours
    bindings:
        - name: cron value: 0 */12 * * *
    annotations:
        - name: feed value: /whisk.system/alarms/alarm

 Rules
* rule: helloworldEvery12Hours
    - trigger: Every12Hours
    - action: helloworld

Do you really want to deploy this? (y/N): y
Deploying package helloworld ... Done!
Deploying action helloworld/helloworld ... Done!
Deploying trigger feed Every12Hours ...
Done!
Deploying rule helloworldEvery12Hours ... Done!

Deployment completed successfully.
```

