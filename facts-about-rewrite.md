

In the last we could dp some thing like this in `apiVersion: extensions/v1beta1` for rewrite, but this is not avaialbe in traefix 2.x!



```s
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: my-ingress-traefik
  annotations:
    kubernetes.io/ingress.class: traefik
    traefik.ingress.kubernetes.io/rewrite-target: /app/
spec:
  rules:
  - host: app.my-domain.fr
    http:
      paths:
      - backend:
          serviceName: my-app
          servicePort: http

```

I treid this ..


```s
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: website-ingress
  labels:
    app: argocdrocks
  annotations:
    ingress.kubernetes.io/ssl-redirect: "false"
    traefik.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: website-service
            port: 
              number: 80
```

This is not suported!

With the new core notions of v2 (introduced earlier in the section "Frontends and Backends Are Dead... Long Live Routers, Middlewares, and Services"), transforming the URL path prefix of incoming requests is configured with middlewares, after the routing step with router rule PathPrefix.

Use Case: Incoming requests to http://example.org/admin are forwarded to the webapplication "admin", with the path /admin stripped, e.g. to http://<IP>:<port>/. In this case, you must:

First, configure a router named admin with a rule matching at least the path prefix with the PathPrefix keyword,

Then, define a middleware of type stripprefix, which removes the prefix /admin, associated to the router admin.

Example ... this is just a sample of how I learned how to create the correct ingress route + middleware

```yaml
---
apiVersion: traefik.containo.us/v1alpha1
kind: IngressRoute
metadata:
  name: http-redirect-ingressroute
  namespace: admin-web
spec:
  entryPoints:
    - web
  routes:
    - match: Host(`example.org`) && PathPrefix(`/blue`)
      kind: Rule
      services:
        - name: admin-svc
          port: admin
      middlewares:
        - name: admin-stripprefix
---
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: admin-stripprefix
spec:
  stripPrefix:
    prefixes:
      - /admin
```