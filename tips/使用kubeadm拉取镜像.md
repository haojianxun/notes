# 使用kubeadm拉取镜像

```
kubeadm config images pull
```

选项

```
Flags:
      --config string               Path to kubeadm config file.
      --cri-socket-path string      Path to the CRI socket. (default "/var/run/dockershim.sock")
      --feature-gates string        A set of key=value pairs that describe feature gates for various features. Options are:
                                    Auditing=true|false (ALPHA - default=false)
                                    CoreDNS=true|false (default=true)
                                    DynamicKubeletConfig=true|false (ALPHA - default=false)
                                    SelfHosting=true|false (ALPHA - default=false)
                                    StoreCertsInSecrets=true|false (ALPHA - default=false)
  -h, --help                        help for pull
      --kubernetes-version string   Choose a specific Kubernetes version for the control plane. (default "stable-1.11")

Global Flags:
      --kubeconfig string   The KubeConfig file to use when talking to the cluster. (default "/etc/kubernetes/admin.conf")
  -v, --v Level             log level for V logs
```

