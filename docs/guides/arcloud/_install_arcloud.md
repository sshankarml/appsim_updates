### Install Certificate Manager

```shell
export CERT_MANAGER_VERSION=1.9.1
```

```shell showLineNumbers
helm upgrade --install --wait --repo https://charts.jetstack.io cert-manager cert-manager \
  --version ${CERT_MANAGER_VERSION} \
  --create-namespace \
  --namespace cert-manager \
  --set installCRDs=true
```

```shell
kubectl -n istio-system apply -f ./setup/issuer.yaml
```

```shell
cat ./setup/certificate.yaml | envsubst | kubectl -n istio-system apply -f -
```

### Create K8s Namespace

```shell showLineNumbers
kubectl create namespace ${NAMESPACE}
kubectl label namespace ${NAMESPACE} istio-injection=enabled
```

### Create Container Registry Secret

```shell showLineNumbers
kubectl --namespace ${NAMESPACE} delete secret container-registry --ignore-not-found
kubectl --namespace ${NAMESPACE} create secret docker-registry container-registry \
  --docker-server=${REGISTRY_SERVER} \
  --docker-username=${REGISTRY_USERNAME} \
  --docker-password=${REGISTRY_PASSWORD}
```

#### Log in to the container registry

```shell
docker login ${REGISTRY_SERVER} --username "${REGISTRY_USERNAME}" --password "${REGISTRY_PASSWORD}"
```

### Setup AR Cloud

```shell showLineNumbers
./setup.sh \
  --set global.domain=${DOMAIN} \
  --no-secure \
  --no-observability \
  --accept-sla
```

:::note Software License Agreement
Passing the `--accept-sla` flag assumes the acceptance of the [Magic Leap 2 Software License Agreement](https://www.magicleap.com/software-license-agreement-ml2).
:::
