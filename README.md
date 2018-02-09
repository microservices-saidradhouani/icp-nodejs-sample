# Node.js sample Helm Chart

### This sample is for demonstrative purposes only and is NOT for production use. ###

### For all features to be available this is best viewed in Google Chrome or Safari with client-side JavaScript enabled.

### Image scan results
Be aware that, as with anything you deploy using Docker, this image may become vulnerable over time; as can your applications!
it's your responsibility to ensure your images are free of problems: Dockerhub and various IBM offerings including the IBM Container Service allow for on-demand image scanning.

## Introduction
This sample application is intended to guide you through the process of deploying your own Node.js applications into IBM Cloud Private. Useful links and examples are provided and the application itself is one that makes use of various monitoring capabilities. Note that this sample was produced in early October 2017 and so the code you are provided with by using the generators may differ! The application provided uses the node Docker images.

This sample was created using `idt create` with the following choices:
- Web App
- Basic Web
- Node

Modifications were then made to use EJS, to add a gulp task, and to add the content. The stylesheet provided is largely based on the [Node.js @ IBM developer center](https://developer.ibm.com/node).

- This example uses [appmetrics](https://github.com/RuntimeTools/appmetrics) and [appmetrics-dash](https://github.com/RuntimeTools/appmetrics-dash): the endpoint being `/appmetrics-dash`.
- This example features the "scrape" annotation in the `<chart directory>/templates/service.yaml` file. In combination with the [appmetrics-prometheus](https://github.com/RuntimeTools/appmetrics-prometheus) module inclusion and usage, this enables the sample to be automatically scraped by a deployed instance of Prometheus in order for metrics to be gathered and displayed using the Prometheus web UI. You can view the raw data that will be available to Prometheus at the `/metrics` endpoint.
This allows developers to quickly determine how the application is performing across potentially many Kubernetes pods.
- This example uses [appmetrics-zipkin](https://github.com/RuntimeTools/appmetrics-zipkin). If Zipkin is deployed (e.g. with the Microservice Builder fabric), trace information will be available under the service name "icp-nodejs-sample". To enable this feature, modify `Dockerfile` and set `USE_ZIPKIN`. You can dynamically modify applications as well using the IBM Cloud Private web UI - this includes the setting of environment variables and it's recommended you restart the pod for the change to take effect.
- This example can be deployed using the IBM Cloud Developer Tools, for example: `idt deploy -t container --deploy-image-target mycluster.icp:8500/default/nodejs-sample`.

The `mycluster.icp` example here should match up with the entry you've added in `/etc/hosts`: it is the location of the private registry.

## Prerequisites

As well as having a Kubernetes cluster you can deploy to (such as Kubernetes included with IBM Cloud Private), you should have Prometheus deployed into your Kubernetes cluster where this sample will be installed. This is not a mandatory step and can be done after deployment.
Optionally you can set up a Zipkin server (for example, as part of the Microservice Builder Fabric or a standalone Zipkin image), which is where you can configure the sample to use the `appmetrics-zipkin` module, to discover the Zipkin server, and to send its trace data there. This is especially useful for end-to-end tracing of your Node.js deployments and more information should be consulted from the [appmetrics-zipkin Github repository](https://github.com/RuntimeTools/appmetrics-zipkin).

## Installing the Chart

The Helm chart can be installed from the app center by clicking the ibm-nodejs-sample tile in the IBM Cloud Private catalog and following the installation steps.

If you don't have the `ibm-charts` repository already available in your install of IBM Cloud Private, or you want to install the sample without using IBM Cloud Private, you can do the following:
- `helm repo add ibm-charts https://raw.githubusercontent.com/IBM/charts/master/repo/stable/`
- `helm install --name sample ibm-charts/ibm-nodejs-sample`

The Helm chart can also be installed with the following command from the directory containing `Chart.yaml`:

`helm install --name tester .` where tester can be anything.

You can find more information about deployment methods in the [IBM Cloud Private documentation](https://www.ibm.com/support/knowledgecenter/SSBS6K/product_welcome_cloud_private.html).

## Verifying the Chart
You can view the deployed sample in your web browser. To retrieve the IP and port:

`export SAMPLE_NODE_IP=$(kubectl get nodes --namespace {{ .Release.Namespace }} -o jsonpath="{.items[0].status.addresses[0].address}")`

`export SAMPLE_NODE_PORT=$(kubectl get --namespace {{ .Release.Namespace }} -o jsonpath="{.spec.ports[0].nodePort}" services {{ template "fullname" . }})`

Open your web browser at `http://${SAMPLE_NODE_IP}:${SAMPLE_NODE_PORT}` to view the sample.

## Uninstalling the Chart

If you installed it with `helm install --name tester .` you'd remove the sample with `helm delete --purge tester`. You can find the deployment with `helm list --all` and searching for an entry with the chart name "ibm-nodejs-sample".

## Testing the Chart with Helm

You can programatically run the test in the following ways.
- `cd chart/ibm-nodejs-sample` then do `./test-chart.sh` OR
- `helm test sample`: assuming you've deployed it with the release name `sample`.

## Configuration

The following table lists the configurable parameters of the ibm-nodejs-sample chart and their default values.

| Parameter                  | Description                                     | Default                                                    |
| -----------------------    | ---------------------------------------------   | ---------------------------------------------------------- |
| `replicaCount`             | How many pods to deploy                         | `1`                                                          |
| `revisionHistoryLimit`     | Optional field that specifies the number of old ReplicaSets to retain to allow rollback   | `1`                |
| `image.repository`         | image repository                                | `ibmcom/icp-nodejs-sample`                                 |
| `image.tag`                | Image tag                                       | `latest`                                                    |
| `image.pullPolicy`         | Image pull policy                               | `Always`                                                   |
| `livenessProbe.initialDelaySeconds`   | How long to wait before checking the pod(s) are up |   `30`                             |
| `livenessProbe.periodSeconds`         | The interval at which we'll check if a pod is running OK before being restarted     | `10`          |
| `service.name`             | Kubernetes service name                                | `Node`                                                     |
| `service.type`             | Kubernetes service type for exposing port                  | `NodePort`                                                 |
| `service.port`             | TCP port for this service                       | `3000`                                                       |
| `resources.limits.memory`  | Memory resource limits                          | `128m`                                                     |
| `resources.limits.cpu`     | CPU resource limits                             | `100m`                                                     |

#### Configuring Node.js applications

See the [Node.js @ IBM developer center](https://developer.ibm.com/node/) for all things Node.js - including more samples, tutorials and blog posts. For configuring Node.js itself, consult the official [Node.js community documentation](https://nodejs.org/en/docs/).

### Deploying on platforms other than x86-64
- Multiarch images are used so the correct Node.js Docker image will be pulled based on your platform. Supported platforms for this sample include ppc64le, x86-64 and s390x.
- Note that the IBM Cloud Developer Tools are not available for every platform: consult the [CLI docs](https://www.ibm.com/cloud/cli) to find out more.

#### Multiple Node.js versions
- This sample is built on the latest LTS version of Node.js.
- Developers are welcome to bring Node.js LTS release applications to IBM Cloud Private and can do so by modifying their Dockerfile to pull from the specified tag that corresponds with the desired version.

#### Logging
This sample involves printing to the standard output stream via the provided `console.log` method in Node.js: as you would with your own deployed Node.js applications. 
You can view these logs in the IBM Cloud Private web UI for this particular deployment and also by using the built-in logging service of IBM Cloud Private itself. One example would be to use Kibana to check what the "log" field contains for anything that comes from logstash. For more information consult the [IBM Cloud Private logging docs](https://www.ibm.com/support/knowledgecenter/en/SSBS6K_2.1.0/manage_metrics/logging_elk.html).

#### Metering
This sample involves metering as defined in the `chart/ibm-nodejs-sample/templates/deployment.yaml` file. You can view the details of any deployment that features metering by using the IBM Cloud Private web UI: it provides a simple way to track what has been deployed into your environment.

### Disclaimers
Node.js is an official trademark of Joyent. Images are used according to the Node.js visual guidelines - no copyright claims are made. You can view the guidelines [here](https://nodejs.org/static/documents/foundation-visual-guidelines.pdf).

This icp-nodejs-sample is not formally related to or endorsed by the official Node.js open source or commercial project.
