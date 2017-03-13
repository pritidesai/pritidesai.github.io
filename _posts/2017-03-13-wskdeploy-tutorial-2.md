In [previous post](http://pritidesai.github.io/2017/02/28/openwhisk-manifest-and-deployment.html) we learnt how to write manifest and deployment files for OpenWhisk WskDepoly. Lets look at how can we build upon it.

## Adding Parameters to Actions:

```bash
function main(params) {
  return {payload:  'Hello, ' + params.name + ' from ' + params.place};
}
```
### manifest.yaml

`
package:
    name: helloworld
    actions:
        helloworld:
            location: src/helloworld.js
            runtime: nodejs
`
**
`
			inputs:
                name:
                    type: string
                    description: name of a person
                place:
                    type: string
                    description: location of a person
`
**
`
			outputs:
                payload:
                    type: string
                    description: a simple greeting message, Hello World!
`



