# Getting started with Whisk Deploy

When you are getting familiar with OpenWhisk by writing hello world action with *helloworld.js*

```bash
/**
 * Return a simple hello world message.
 */
function main(params) {
    return {payload:  'Hello World!'};
}
```

Now if you like to automate deployment of hello world action, continue reading through this article. I am going to show you what all you need to use [openwhisk-wskdeploy](https://github.com/openwhisk/openwhisk-wskdeploy) to automate deployment.

wskdeploy takes two files, (1) manifest file (2) deployment file.

## manifest.yaml

```yaml
package:
    name: helloworld
    version: 1.0
    license: Apache-2.0
    actions:
        helloworld:
            version: 2.0
            location: src/helloworld.js
            runtime: nodejs
            outputs:
                payload:
                    type: string
                    description: a simple greeting message, Hello World!
```

## deployment.yaml

```yaml
application:
    name: SampleHelloWorld
    namespace: _
    version: 0.0.1
    package:
        name: helloworld
        actions:
            helloworld:
```

## directory structure

```bash
ls -1R ~/SampleHelloWorldApp/
deployment.yaml
manifest.yaml
src/

./src:
helloworld.js
```

## Deploy Hello World

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


Triggers:

 Rules

Do you really want to deploy this? (y/N): y
Deploying pacakge helloworld ... Done!
Deploying action helloworld/helloworld ... Done!

Deployment completed successfully.
```

## Verify

```bash
wsk action invoke --blocking --result helloworld/helloworld
{
    "payload": "Hello World!"
}
```

