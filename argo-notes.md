# Argo Notes

I set up a Nodejs test runner Dockerfile [here](https://github.com/dictybase-playground/argo-test/blob/develop/node-test-runner/Dockerfile). 
It is on the [Docker Hub](https://cloud.docker.com/u/dictybase/repository/docker/dictybase/argotest) with the `dev` tag.

After some revisions, the Dockerfile now has a `CMD` of `npm`, which  allows the 
workflow config to give it an argument of `test`. This will trigger `npm test` and 
put the log of this command in the automatically generated workflow pod (i.e. `github-27cqv`).

This works with the defined event source and sensor configurations listed below.

However, there is one glaring issue -- the workflow-provided environmental variables 
(`GITHUB_CLONE_URL` and `GITHUB_REPO_NAME`) are not overriding the default envs 
set inside the Dockerfile. So every time the workflow is triggered, it just runs the 
same tests for `dicty-frontpage`, which is the default for the env inside the Dockerfile.

Perhaps this could be fixed with a multi-stage build? It doesn't appear that you 
can pass multiple `command` keys to the same `template` in a Workflow config, otherwise I would suggest 
maybe having multiple `CMD`s in the Dockerfile, one for `git clone`, one for `npm`, etc. and 
then refer to those in the Workflow. That would allow the ENV to be passed as a parameter instead.

This example seems to be really close to having the desired output, but the Dockerfile 
needs to be able to handle all arguments and environmental variables defined in a Workflow.

### event-source

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: github-event-source
  labels:
    # do not remove
    argo-events-event-source-version: v0.10
data:
  dicty-stock-center: |-
    id: 118239462
    owner: "dictyBase"
    repository: "Dicty-Stock-Center"
    hook:
     endpoint: "/github/dicty-stock-center"
     port: "12000"
     url: "https://ericargo.dictybase.dev"
    events:
    - "push"
    apiToken:
      name: github-argotest
      key: apiToken
    webHookSecret:
      name: github-argotest
      key: webHookSecret
    insecure: false
    active: true
    contentType: "json"
```

### sensor

Note: this is set up `inline` for now to make it easier to test. Can easily be 
switched to `url` later.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Sensor
metadata:
  name: github-sensor
  labels:
    # sensor controller with instanceId "argo-events" will process this sensor
    sensors.argoproj.io/sensor-controller-instanceid: argo-events
    # sensor controller will use this label to match with its own version
    # do not remove
    argo-events-sensor-version: v0.10
spec:
  template:
    spec:
      containers:
        - name: "sensor"
          image: "argoproj/sensor"
          imagePullPolicy: Always
      serviceAccountName: argo-events-sa
  dependencies:
    # name matching event sensor
    - name: "github-gateway:dicty-stock-center"
  eventProtocol:
    type: "HTTP"
    http:
      port: "9300"
  triggers:
    - template:
        name: github-workflow-trigger
        group: argoproj.io
        version: v1alpha1
        kind: Workflow
        source:
          inline: |
            apiVersion: argoproj.io/v1alpha1
            kind: Workflow
            metadata:
              generateName: github-
            spec:
              # name of template to invoke
              entrypoint: argotest
              arguments:
                parameters:
                - name: clone-url
                  value: THIS_WILL_BE_REPLACED
                - name: name
                  value: THIS_WILL_BE_REPLACED
              templates:
              - name: argotest
                inputs:
                  parameters:
                  - name: clone-url
                  - name: name
                container:
                  image: dictybase/argotest:dev
                  command: [npm]
                  args: ["test"]
                  env:
                  - name: GITHUB_CLONE_URL
                    value: "{{inputs.parameters.clone-url}}"
                  - name: GITHUB_REPO_NAME
                    value: "{{inputs.parameters.name}}"
      resourceParameters:
        - src:
            event: "github-gateway:dicty-stock-center"
            path: "head_commit.url"
            # value:
          dest: spec.arguments.parameters.0.value
        - src:
            event: "github-gateway:dicty-stock-center"
            path: "repository.name"
            # value:
          dest: spec.arguments.parameters.1.value
```
