In [Getting Started with Whisk Deploy](https://medium.com/openwhisk/getting-started-with-whisk-deploy-cea744222585#.vv018i65l), we saw how to write manifest and deployment files to deploy [simple hello world action](https://github.com/openwhisk/openwhisk/blob/master/docs/actions.md#creating-and-invoking-javascript-actions). Lets look at how we can add parameters to an action and set defaults using deployment file.

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
