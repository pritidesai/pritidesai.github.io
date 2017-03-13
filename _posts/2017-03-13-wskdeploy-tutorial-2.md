In [Getting Started with Whisk Deploy](https://medium.com/openwhisk/getting-started-with-whisk-deploy-cea744222585#.vv018i65l), we saw how to write manifest and deployment files to deploy [simple hello world action](https://github.com/openwhisk/openwhisk/blob/master/docs/actions.md#creating-and-invoking-javascript-actions). Lets look at how we can add parameters to the same action, set defaults using deployment file, define trigger and rule.

## Adding Parameters to Actions:

```bash
function main(params) {	
  return {payload:  'Hello, ' + params.name + ' from ' + params.place};
}
```

### manifest.yaml

Define parameters by adding **inputs** section under action name **"helloworld"**:

```yaml
package:
    name: helloworld
    actions:
        helloworld:
            location: src/helloworld.js
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

### deployment.yaml

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

### Same directory structure as before

```bash
ls -1R ~/SampleHelloWorldApp/
deployment.yaml
manifest.yaml
src/

./src:
helloworld.js
```

### Deploy Hello World

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

### Verify

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

### manifest.yaml

Add a section called "triggers" and "rules" 

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

### Deploy Hello World action, trigger, and rule:

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

### Verify

#### Poll for running actions:

```bash
wsk activation poll
Enter Ctrl-c to exit.
Polling for activation logs
```

#### Fire trigger:

Open one more terminal and fire the trigger:

```bash
wsk trigger fire locationUpdate
ok: triggered /pdesai@us.ibm.com_dev/locationUpdate with id 4c3a8b1792d546a68ac58538c3f5d637
```

#### Result from polling:

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

#### Determine activation ID from polling and use that to get action result: 

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

