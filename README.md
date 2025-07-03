# Split Helm Charts

This repository contains a collection of Helm charts for deploying Split services in Kubernetes:

- **Split Evaluator**: For server-side evaluation of feature flags
- **Split Proxy**: For client-side SDKs to communicate with Split
- **Split Synchronizer**: For synchronizing Split data with Redis

These charts are based on the setups described in [Kubernetes and Split](https://www.harness.io/blog/kubernetes-and-split).


## Usage

### Basic Installation

To deploy a service, use the appropriate chart with `helm install` and provide your Split API key:

```shell
# Split Evaluator
helm install my-evaluator ./split-evaluator --set config.apiKey=<yourApiKey>

# Split Proxy
helm install my-proxy ./split-proxy \
  --set config.apiKey=<yourApiKey> \
  --set config.clientApiKeys=<yourClientKeys>

# Split Synchronizer
helm install my-sync ./split-synchronizer --set apiKey=<yourApiKey>
```

### Customizing Deployments

Customize deployments by providing values for chart parameters using the `--set` flag:

```shell
# Example: Setting replica count for the Split Evaluator
helm install my-evaluator ./split-evaluator \
  --set replicaCount=5 \
  --set config.apiKey=<yourApiKey>
```

You can also use a values file for more complex configurations:

```shell
helm install my-evaluator ./split-evaluator -f my-values.yaml
```

For more information on customizing Helm deployments, refer to the [Helm documentation](https://helm.sh/docs/).

## Using Sealed Secrets for API Keys

All charts in this repository now support storing API keys in [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets) instead of ConfigMaps for enhanced security. This provides better protection for sensitive API keys in Kubernetes.

### Why Use Sealed Secrets?

- **Enhanced Security**: API keys are encrypted and can only be decrypted by the controller running in your cluster
- **GitOps Friendly**: Encrypted secrets can be safely stored in Git repositories
- **Access Control**: Limits who can access the actual API key values

### Prerequisites

1. **Install the sealed-secrets controller** in your cluster:

```shell
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm repo update
helm install sealed-secrets sealed-secrets/sealed-secrets --namespace kube-system
```

2. **Install the `kubeseal` CLI tool**:

```shell
# On macOS
brew install kubeseal

# On Linux
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.19.5/kubeseal-0.19.5-linux-amd64.tar.gz
tar -xvzf kubeseal-0.19.5-linux-amd64.tar.gz kubeseal
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
```

### Creating Sealed Secrets

#### For Split Proxy

```shell
# 1. Create a regular Kubernetes Secret with your API keys
kubectl create secret generic split-proxy-sealed \
  --from-literal=apiKey=your-split-api-key \
  --from-literal=clientApiKeys=your-client-api-keys \
  --dry-run=client -o yaml > proxy-secret.yaml

# 2. Use kubeseal to encrypt the secret
kubeseal < proxy-secret.yaml > proxy-sealed-secret.yaml

# 3. Apply the sealed secret to your cluster
kubectl apply -f proxy-sealed-secret.yaml
```

#### For Split Evaluator

```shell
# Create and seal the secret
kubectl create secret generic split-evaluator-sealed \
  --from-literal=apiKey=your-evaluator-api-key \
  --dry-run=client -o yaml | kubeseal > evaluator-sealed-secret.yaml

# Apply the sealed secret
kubectl apply -f evaluator-sealed-secret.yaml
```

#### For Split Synchronizer

```shell
# Create and seal the secret
kubectl create secret generic split-synchronizer-sealed \
  --from-literal=apiKey=your-synchronizer-api-key \
  --dry-run=client -o yaml | kubeseal > synchronizer-sealed-secret.yaml

# Apply the sealed secret
kubectl apply -f synchronizer-sealed-secret.yaml
```

### Deploying Charts with Sealed Secrets

#### Split Proxy

```shell
helm install my-proxy ./split-proxy \
  --set config.useSealed=true \
  --set config.sealed.secretName=split-proxy-sealed
```

#### Split Evaluator

```shell
helm install my-evaluator ./split-evaluator \
  --set config.useSealed=true \
  --set config.sealed.secretName=split-evaluator-sealed
```

#### Split Synchronizer

```shell
helm install my-synchronizer ./split-synchronizer \
  --set useSealed=true \
  --set sealed.secretName=split-synchronizer-sealed
```

### Customizing Secret Names and Keys

You can customize the secret name and key names within the secret:

```shell
# For Split Proxy
helm install my-proxy ./split-proxy \
  --set config.useSealed=true \
  --set config.sealed.secretName=my-custom-secret-name \
  --set config.sealed.apiKeyKey=my-api-key \
  --set config.sealed.clientApiKeysKey=my-client-keys

# For Split Evaluator
helm install my-evaluator ./split-evaluator \
  --set config.useSealed=true \
  --set config.sealed.secretName=custom-evaluator-secret \
  --set config.sealed.apiKeyKey=custom-api-key-name

# For Split Synchronizer
helm install my-synchronizer ./split-synchronizer \
  --set useSealed=true \
  --set sealed.secretName=custom-sync-secret \
  --set sealed.apiKeyKey=custom-api-key-name
```

### Generating Template References

You can generate template references using the `--dry-run` flag and `createSealedSecretTemplate` option:

```shell
# For Split Proxy
helm install my-proxy ./split-proxy \
  --set config.useSealed=true \
  --set config.createSealedSecretTemplate=true \
  --dry-run

# For Split Evaluator
helm install my-evaluator ./split-evaluator \
  --set config.useSealed=true \
  --set config.createSealedSecretTemplate=true \
  --dry-run

# For Split Synchronizer
helm install my-sync ./split-synchronizer \
  --set useSealed=true \
  --set createSealedSecretTemplate=true \
  --dry-run
```

### Important Notes

- **Create the secret first**: Always create and apply the sealed secret before installing the chart
- **Secret names must match**: The secret name in your chart configuration must match the actual sealed secret name
- **Key names must match**: The key names in your chart configuration must match the keys in your sealed secret
- **No automatic secret creation**: The chart does not create sealed secrets; you must create them separately

## Integrating with the Split SDKs

The [Split Evaluator](https://help.split.io/hc/en-us/articles/360020037072-Split-Evaluator) does not need Split SDKs and can be integrated with via HTTP API calls. 

The [Split Proxy](https://help.split.io/hc/en-us/articles/4415960499213-Split-Proxy) requires Split SDKs to be configured to use the events Service as the events endpoint of the SDK and the sdk service as the sdk endpoint of the SDK. See this split help page for more info: [How to use Split SDKs with Split Proxy](https://help.split.io/hc/en-us/articles/360053243551-General-SDK-How-to-use-Split-SDKs-with-Split-Proxy)

The [Split Synchronizer](https://help.split.io/hc/en-us/articles/360019686092-Split-Synchronizer) requires specific configuration for each Split SDK to read from the Synchronizer and be set to use Consumer mode. Please review the configuration options for your Server Side SDK listed in the Split Help Center here: [Server Side SDKs](https://help.split.io/hc/en-us/sections/12619253757069-Server-side-SDKs)

## License

This repository is licensed under the [MIT License](LICENSE).
