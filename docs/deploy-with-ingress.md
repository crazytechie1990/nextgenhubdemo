# Deploy with ingress

The `hello-kubernetes` Helm chart can be used to deploy and configure the `hello-kubernetes` application for use with an ingress controller. 

> **Note:**
> 
> The `hello-kubernetes` Helm chart does **not** deploy an [Ingress Controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) and does **not** deploy the [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) definition.
>
> The chart aims to support deployment to as many platforms and providers as possible, so the choice of Ingress Controller and configuration of Ingress resource is left to the person deploying.

## Prerequisites

- [Helm 3](https://v3.helm.sh/)

If you are using the [VS Code Remote Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) based development environment, all of the prerequisites will be available in the terminal.

## Install ingress controller

Install an [Ingress Controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) that is available for your platform or provider. Here is an example that uses the [Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/) on a cloud provider:

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

helm install nginx-ingress ingress-nginx/ingress-nginx \
    --create-namespace --namespace ingress \
    --set controller.replicaCount=2
```

## Use hello-kubernetes with ingress

### Deploy hello-kubernetes instances

Ensure that you are in the chart directory in the repo, since the instructions reference a local helm chart.

```bash
cd deploy/helm
```

Install `hello-kubernetes` instances that will be available via different path on the ingress. 

The `hello-world` instance will display the default "Hello world!" message.

```bash
helm install --create-namespace --namespace hello-kubernetes hello-world ./hello-kubernetes \
  --set ingress.configured=true --set ingress.pathPrefix=hello-world \
  --set service.type=ClusterIP
```

### Deploy ingress definition

The `hello-kubernetes` Helm chart has a `ingress.rewritePath` configuration parameter that is `true` by default. When used together with the `ingress.configured=true` configuration parameter, there is an assumption that the ingress being used supports path rewrites. See the [Deploy using Helm](deploy-using-helm.md) guidance for more details.

So from our example, a request to `/hello-world` should be rewritten to `/` before being passed to the `hello-world` app instance.

Create a file named `hello-kubernetes-ingress.yaml` with the content below. This ingress definition will be serviced by the nginx ingress controller due to the `kubernetes.io/ingress.class: nginx` annotation. It will also leverage the path rewrite capabilities of nginx via the `nginx.ingress.kubernetes.io/rewrite-target: /$2` annotation.

```yaml
# hello-kubernetes-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-kubernetes-ingress
  namespace: hello-kubernetes
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: hello-kubernetes-hello-world
                port:
                  number: 80

```

Deploy the contents of the `hello-kubernetes-ingress.yaml` into the same namespace as the two `hello-kubernetes` apps.

```bash
kubectl apply -n hello-kubernetes -f hello-kubernetes-ingress.yaml
```

### Browse

You can browse to each of the `hello-kubernetes` apps via the $INGRESS_CONTROLLER_IPADDRESS and each of the configured paths. So for our example at:

- `$INGRESS_CONTROLLER_IPADDRESS/hello-world` - the `hello-world` instance with the default "Hello world!" message

## Alternatives

You can deploy the `hello-kubernetes` app via the Helm chart with the `ingress.rewritePath=false` configuration parameter if you are deploying with an ingress controller that does not support path rewrites.

In this case, the `hello-kubernetes` apps will serve dynamic content and static assets from the path defined by the `ingress.pathPrefix` configuration parameter.
