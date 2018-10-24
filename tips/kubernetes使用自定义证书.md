# kubernetes使用自定义证书

默认情况下，kubeadm 将生成集群运行所需的所有证书。您可以通过提供自己的证书来覆盖此行为。

为此，您必须将它们放置在 `--cert-dir` 参数或 `CertificatesDir` 配置文件密钥所指定的目录中。默认目录为 `/etc/kubernetes/pki`。

如果给定的证书和私钥对都存在，kubeadm 会跳过生成步骤，这些文件将被验证并用于规定的用例。

这意味着你可以用一个现有的 CA 预填充 `/etc/kubernetes/pki/ca.crt` 和 `/etc/kubernetes/pki/ca.key`，然后用它来签署其余的证书。