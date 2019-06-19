# Argo Notes

I set up a Nodejs test runner Dockerfile [here](https://github.com/dictybase-playground/argo-test/blob/develop/node-test-runner/Dockerfile). 
It is on the [Docker Hub](https://cloud.docker.com/u/dictybase/repository/docker/dictybase/argotest) with the `dev` tag.

It is my understanding that you cannot pass defined arguments through the Argo 
workflow. Because of this, I have updated the Dockerfile to use `ENV`. This seems 
to work if the `ENV` in the Dockerfile is also given default values. Here is [an example](https://ericargo.dictybase.dev/workflows/argo-events/github-wx4dd?tab=workflow&nodeId=github-wx4dd) 
of a workflow that succeeded with this format.

However, nothing is being logged this way. It does not show the logs from the 
Docker container being built. Need to determine a way to see the output of the 
tests being ran during this build.

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
  dev-notes: |-
    # ID of the GitHub webhook
    # this needs to match the one you generated
    id: 117982148
    owner: "dictybase-playground"
    repository: "dev-notes"
    # Github will send events to the following port and endpoint
    hook:
     endpoint: "/github/dev-notes"
     port: "12000"
     # url the gateway will use to register at GitHub
     url: "https://ericargo.dictybase.dev"
    # type of events to listen to
    events:
    - "*"
    # apiToken refers to K8s secret that stores the github personal access token
    apiToken:
      # Name of the K8s secret that contains the access token
      name: github-access
      # Corresponding key in the K8s secret
      key: apiToken
    # webHookSecret refers to K8s secret that stores the webhook secret
    webHookSecret:
      name: github-access
      key: webHookSecret
    # type of connection between gateway and github
    insecure: false
    # determines if notifications are sent when the webhook is triggered
    active: true
    contentType: "json"

  argotest: |-
    id: 117990339
    owner: "dictybase-playground"
    repository: "argo-test"
    hook:
     endpoint: "/github/argo-test"
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

  dicty-stock-center: |-
    id: 118239400
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
    # - name: "github-gateway:dev-notes"
    # - name: "github-gateway:argotest"
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
                  env:
                  - name: GITHUB_CLONE_URL
                    value: "{{inputs.parameters.clone-url}}"
                  - name: GITHUB_REPO_NAME
                    value: "{{inputs.parameters.name}}"
      resourceParameters:
        # - src:
        #     event: "github-gateway:dev-notes"
        #   dest: spec.arguments.parameters.0.value
        - src:
            event: "github-gateway:dicty-stock-center"
            path: "repository.clone_url"
            # value:
          dest: spec.arguments.parameters.0.value
        - src:
            event: "github-gateway:dicty-stock-center"
            path: "repository.name"
            # value:
          dest: spec.arguments.parameters.1.value
```

Also needed to set up clusterrole for argo service account.

`kubectl create clusterrolebinding argo-events --clusterrole=cluster-admin --serviceaccount=argo-events:default`
