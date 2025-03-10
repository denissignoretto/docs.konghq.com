---
title: Configuring a fallback service
---

This guide walks through how to setup a fallback service using Ingress
resource. The fallback service will receive all requests that don't
match against any of the defined Ingress rules.
This can be useful for scenarios where you would like to return a 404 page
to the end user if the user clicks on a dead link or inputs an incorrect URL.

## Installation

Please follow the [deployment](/kubernetes-ingress-controller/{{page.kong_version}}/deployment/overview) documentation to install
the {{site.kic_product_name}} on your Kubernetes cluster.

## Testing Connectivity to Kong

This guide assumes that the `PROXY_IP` environment variable is
set to contain the IP address or URL pointing to Kong.
Please follow one of the
[deployment guides](/kubernetes-ingress-controller/{{page.kong_version}}/deployment/overview) to configure this environment variable.

If everything is setup correctly, making a request to Kong should return
HTTP 404 Not Found.

```bash
$ curl -i $PROXY_IP
HTTP/1.1 404 Not Found
Date: Fri, 21 Jun 2019 17:01:07 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Content-Length: 48
Server: kong/1.2.1

{"message":"no Route matched with those values"}
```

This is expected as Kong does not yet know how to proxy the request.

## Setup a Sample Service

For the purpose of this guide, we will setup a simple HTTP service in the
cluster and proxy it.

```bash
$ echo '
apiVersion: apps/v1
kind: Deployment
metadata:
  name: fallback-svc
spec:
  replicas: 1
  selector:
    matchLabels:
      app: fallback-svc
  template:
    metadata:
      labels:
        app: fallback-svc
    spec:
      containers:
      - name: fallback-svc
        image: hashicorp/http-echo
        args:
        - "-text"
        - "This is not the path you are looking for. - Fallback service"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: fallback-svc
  labels:
    app: fallback-svc
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 5678
    protocol: TCP
    name: http
  selector:
    app: fallback-svc
' | kubectl apply -f -
```

Result:

```bash
deployment.apps/fallback-svc created
service/fallback-svc created
```

Create an Ingress rule to proxy the httpbin service we just created:

```bash
$ echo '
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: demo
  annotations:
    konghq.com/strip-path: "true"
spec:
  ingressClassName: kong
  rules:
  - http:
      paths:
      - path: /foo
        pathType: ImplementationSpecific
        backend:
          service:
            name: httpbin
            port:
              number: 80
' | kubectl apply -f -
ingress.extensions/demo created
```

Test the Ingress rule:

```bash
$ curl -i $PROXY_IP/foo/status/200
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 0
Connection: keep-alive
Server: gunicorn/19.9.0
Date: Wed, 17 Jul 2019 19:25:32 GMT
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true
X-Kong-Upstream-Latency: 2
X-Kong-Proxy-Latency: 1
Via: kong/1.2.1
```

## Setup a fallback service

Let's deploy another sample service service:

```bash
$ kubectl apply -f https://bit.ly/fallback-svc
deployment.extensions/fallback-svc created
service/fallback-svc created
```

Next, let's set up an Ingress rule to make it the fallback service
to send all requests to it that don't match any of our Ingress rules:

```bash
$ echo "
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: fallback
  annotations:
spec:
  ingressClassName: kong
  defaultBackend:
    service:
      name: fallback-svc
      port:
        number: 80
" | kubectl apply -f -
```

## Test it

Now send a request with a request property that doesn't match against
any of the defined rules:

```bash
$ curl $PROXY_IP/random-path
This is not the path you are looking for. - Fallback service
```

The above message comes from the fallback service that was deployed in the
last step.

Create more Ingress rules, some complicated regex based ones and
see how requests that don't match any rules, are forwarded to the
fallback service.

You can also use Kong's request-termination plugin on the `fallback`
Ingress resource to terminate all requests at Kong, without
forwarding them inside your infrastructure.
