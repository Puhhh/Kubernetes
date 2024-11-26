# Kubernetes Cluster Step By Step

## Talos OS Configure

### Control plane

```bash
talosctl gen secrets -o secrets.yaml
talosctl gen config --with-secrets secrets.yaml k8s-test https://172.168.101.100:6443
```

```bash
talosctl apply-config --insecure -n 172.168.101.100 --file controlplane.yaml
talosctl bootstrap -n 172.168.101.100 -e 172.168.101.100  --talosconfig=./talosconfig
talosctl kubeconfig -n 172.168.101.100 -e 172.168.101.100 --talosconfig=./talosconfig
```

### Workers

```bash
talosctl apply-config --insecure -n 172.168.101.101 --file worker.yaml --talosconfig=./talosconfig
talosctl apply-config --insecure -n 172.168.101.102 --file worker.yaml --talosconfig=./talosconfig
```

## Kubernetes Configure

[terraform-k8s-apps](https://github.com/Puhhh/terraform-k8s-apps)

[terragrunt-k8s-apps](https://github.com/Puhhh/terragrunt-k8s-apps)
