# kubernetes中进入pod运行命令

```
kubectl exec POD [-c CONTAINER] -- COMMAND [args...] [options]

例如:
kubectl exec pod-demo -c myapp  -- /bin/sh -it
```

