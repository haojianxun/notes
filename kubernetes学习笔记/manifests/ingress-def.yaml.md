# ingress-def.yaml

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPlicy
metadata:
  name: allow-all-ingress
spec:
 podSelector: {}
 ingress:
 - {}
 plicyTypes:
 - Ingress
```

