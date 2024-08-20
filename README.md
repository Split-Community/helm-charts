# Helm Charts

This repository contains a collection of Helm charts that can be used to deploy the Split Evaluator, Split Proxy, and Split Synchronizer. These are based off of the setups described in this blog post: [Kubernetes and Split](https://www.split.io/blog/kubernetes-and-split/). These do not include the sample python app used in those examples.

## Installation

To use these Helm charts, you need to have Helm installed on your local machine. If you don't have Helm installed, please refer to the official Helm documentation for installation instructions.

Once Helm is installed, you can add this repository to your Helm repositories by running the following command:

```shell
helm repo add helm-charts https://github.com/split-community/helm-charts
```

## Usage

To deploy an application or service using one of the Helm charts in this repository, you can use the `helm install` command. For example, to deploy the `split-evaluator` chart, run the following command:

```shell
helm install my-evaluator helm-charts/split-evaluator --set config.apiKey=<yourApiKey>
```

You can customize the deployment by providing values for the chart's parameters using the `--set` flag. For example, to set the number of replicas for the `my-evaluator` deployment, run the following command:

```shell
helm install my-proxy helm-charts/my-app --set replicaCount=5 --set config.apiKey=<yourApiKey>
```

For more information on how to use Helm and customize the deployment, please refer to the Helm documentation.

## Integrating with the Split SDKs

The (Split Evaluator)[https://help.split.io/hc/en-us/articles/360020037072-Split-Evaluator] does not need Split SDKs and can be integrated with via HTTP API calls. 

The (Split Proxy)[https://help.split.io/hc/en-us/articles/4415960499213-Split-Proxy] requires Split SDKs to be configured to use the events Service as the events endpoint of the SDK and the sdk service as the sdk endpoint of the SDK. See this split help page for more info: (How to use Split SDKs with Split Proxy)[https://help.split.io/hc/en-us/articles/360053243551-General-SDK-How-to-use-Split-SDKs-with-Split-Proxy]

The (Split Synchronizer)[https://help.split.io/hc/en-us/articles/360019686092-Split-Synchronizer] requires specific configuration for each Split SDK to read from the Synchronizer and be set to use Consumer mode. Please review the configuration options for your Server Side SDK listed in the Split Help Center here: (Server Side SDKs)[https://help.split.io/hc/en-us/sections/12619253757069-Server-side-SDKs]

## License

This repository is licensed under the [MIT License](LICENSE).
