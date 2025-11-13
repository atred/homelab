# homelab-cluster

- Activate direnv
- Install flux
  - Need personal access token from github, copy it
  - Export token as GITHUB_TOKEN and username as GITHUB_USER
  - flux check --pre
  - Below command:
  ```bash
    flux bootstrap github \
      --owner=$GITHUB_USER \
      --repository=homelab \
      --branch=master \
      --path=./clusters/staging \
      --personal
  ```
- Bootstrap sops (TODO improve this)
```sh
cat age.agekey | \
  kubectl create secret generic sops-age \
  --namespace flux-system \
  --from-file=age.agekey=/dev/stdin
```

## Talos Bootstrap
Ensure your age key (can be downloaded from password manager) is available in the right location:
- Linux: `~/.config/sops/age/keys.txt`
- macOS: `"$HOME/Library/Application Support/sops/age/keys.txt"`

1. Download talos iso with the following extensions:
  - siderolabs/i915
  - siderolabs/intel-ucode
  - siderolabs/iscsi-tools
2. Install it on the nodes
3. Run the following:
```
# Bootstrap config
talhelper genconfig
talosctl apply-config --insecure --nodes 10.100.50.11 --file clusterconfig/homelab-cluster-g3m1.yaml
talosctl apply-config --insecure --nodes 10.100.50.12 --file clusterconfig/homelab-cluster-g3m2.yaml
talosctl apply-config --insecure --nodes 10.100.50.13 --file clusterconfig/homelab-cluster-g3m3.yaml
talosctl config endpoints 10.100.50.11 # no need to specify talos config if direnv is working

# Bootstrap cluster
talosctl bootstrap --nodes 10.100.50.11
talosctl config endpoints 10.100.50.10 # After bootstrap, point to VIP
talosctl kubeconfig kubeconfig.yaml --nodes 10.100.50.11

# Check health:
talosctl --nodes 10.100.50.11 health
```
