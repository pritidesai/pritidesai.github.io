This story is a follow up on [Getting Started with Whisk Deploy](https://medium.com/openwhisk/getting-started-with-whisk-deploy-cea744222585#.vv018i65l) where we learnt how to automate deployment of [simple hello world action](https://github.com/openwhisk/openwhisk/blob/master/docs/actions.md#creating-and-invoking-javascript-actions). Lets look at how we can:

* [Pass parameters to an action](https://github.com/openwhisk/openwhisk/blob/master/docs/actions.md#passing-parameters-to-an-action)
* [Set defaults to those parameters using Whisk Deploy](https://github.com/openwhisk/openwhisk/blob/master/docs/actions.md#setting-default-parameters)
* [Create and Deploy trigger](https://github.com/openwhisk/openwhisk/blob/master/docs/triggers_rules.md#creating-and-firing-triggers) 
* [Associate triggers and actions by using rule](https://github.com/openwhisk/openwhisk/blob/master/docs/triggers_rules.md#associating-triggers-and-actions-by-using-rules)

## Passing Parameters to Action:

```bash
cat hello.js
function main(params) {	
  return {payload:  'Hello, ' + params.name + ' from ' + params.place};
}
```

### Step 1: Create a manifest file (manifest.yaml)

Define parameters by adding **inputs** section under action name **"helloworld"**:

```yaml
package:
    name: helloworld
    actions:
        helloworld:
            location: src/hello.js
            runtime: nodejs
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
```

### Step 2: Create a deployment file (deployment.yaml)

Set default parameters by adding **inputs** section under action name **"helloworld"**:

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
```

### Step 3: Same directory structure

```bash
ls -1R ~/SampleHelloWorldApp/
deployment.yaml
manifest.yaml
src/

./src:
hello.js
```

### Step 4: Deploy Hello World Action

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
  * action: helloworld
    bindings:
        - name: name value: Amy
        - name: place value: Paris


Triggers:

 Rules

Do you really want to deploy this? (y/N): y
Deploying pacakge helloworld ... Done!
Deploying action helloworld/helloworld ... Done!

Deployment completed successfully.
```

### Step 5: Verify your deployment

```bash
wsk action invoke --blocking --result helloworld/helloworld
{
    "payload": "Hello, Amy from Paris"
}
```

```
wsk action invoke --blocking --result helloworld/helloworld --param name Mark --param place Barcelona
{
    "payload": "Hello, Mark from Barcelona"
}
```

## Create Trigger and Rule

### Step 1: Update the manifest file (manifest.yaml)

Add a section named "triggers" and "rules": 

```
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
        locationUpdate:															# trigger name
    rules:
        hellowroldOnLocationUpdate:												# rule name
            action: helloworld
            trigger: locationUpdate
```

### Step 2: Deploy Hello World action, trigger, and rule:

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
  * action: helloworld
    bindings:
        - name: name value: Amy
        - name: place value: Paris


Triggers:
* trigger: locationUpdate
    bindings:

 Rules
* rule: hellowroldWithLocationUpdate
    - trigger: locationUpdate
    - action: helloworld

Do you really want to deploy this? (y/N): y
Deploying pacakge helloworld ... Done!
Deploying action helloworld/helloworld ... Done!
Deploying trigger locationUpdate ... Done!
Deploying rule hellowroldWithLocationUpdate ... Done!

Deployment completed successfully.
```

### Step 3: Verify your deployment

#### (1) Poll for running actions:

```bash
wsk activation poll
Enter Ctrl-c to exit.
Polling for activation logs
```

#### (2) Fire trigger:

Open one more terminal and fire the trigger:

```bash
wsk trigger fire locationUpdate
ok: triggered /pdesai@us.ibm.com_dev/locationUpdate with id 4c3a8b1792d546a68ac58538c3f5d637
```

#### (3) Result from polling:

```bash
wsk activation poll
Enter Ctrl-c to exit.
Polling for activation logs

Activation: helloworld (d545c458f3d34d6fbf5c29173be3d29e)
[]

Activation: locationUpdate (4c3a8b1792d546a68ac58538c3f5d637)
[]

Activation: hellowroldWithLocationUpdate (c099355c1f1f4d6d8d30f54e8dac2b84)
[]
```

#### (4) Determine activation ID from polling and get the result of that action: 

```bash
wsk activation get d545c458f3d34d6fbf5c29173be3d29e
ok: got activation d545c458f3d34d6fbf5c29173be3d29e
{
	...
    "activationId": "d545c458f3d34d6fbf5c29173be3d29e",
    "start": 1489444142544,
    "end": 1489444142598,
    "response": {
        "status": "success",
        "statusCode": 0,
        "success": true,
        "result": {
            "payload": "Hello, Amy from Paris"
        }
    },
	...
}
```
