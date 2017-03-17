# Whisk Deploy — Alarm Trigger

OpenWhisk actions can  be triggered periodically by an alarm trigger (similar to cron job). Lets look at how can we deploy alarm trigger using wskdeploy:

## Step 1: Create a manifest file (manifest.yaml)

To define an alarm trigger, we have to specify source of the trigger as /whisk.system/alarms/alarm in manifest.yaml file:
package:

```yaml
    name: helloworld
    actions:
        helloworld:
            location: src/hello.js
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
        Every12Hours:
            source: /whisk.system/alarms/alarm
    rules:
        hellowroldOnCron:
            action: helloworld
            trigger: Every12Hours

```

## Step 2: Create a deployment file (deployment.yaml)

Set cron as an input to alarm trigger Every12Hours: 

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
            Every12Hours:
                inputs:
                    cron: "0 */12 * * *"
```

You can also set trigger payload (action parameter values during each trigger) in deployment file using trigger_payload:
application:

```yaml
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
            Every12Hours:
                inputs:
                    cron: "0 */12 * * * *"
                    trigger_payload: "{\"name\":\"Mark\", \"place\":\"Barcelona\"}"
```

## Step 3: Deploy a sample action

```bash
./wskdeploy -p ~/SampleHelloWorldApp/
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
        - name: cron value: 0 */12 * * * *
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

At this point, we have an action hello world configured to run every 12 hours. We can verify our deployment by firing a trigger and checking if action is invoked by following instructions from Whisk Deploy — Action, Trigger, and Rule.

Enjoy!
