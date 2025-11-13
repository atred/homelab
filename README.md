# homelab-cluster

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
cd talos/
talhelper genconfig
# If you need to change secrets, do it from the .sops.yaml file in the talos dir
cd ..
talosctl apply-config --insecure --nodes 10.100.50.11 --file talos/clusterconfig/homelab-cluster-g3m1.yaml
talosctl apply-config --insecure --nodes 10.100.50.12 --file talos/clusterconfig/homelab-cluster-g3m2.yaml
talosctl apply-config --insecure --nodes 10.100.50.13 --file talos/clusterconfig/homelab-cluster-g3m3.yaml
talosctl config endpoints 10.100.50.11 # no need to specify talos config if direnv is working

# Bootstrap cluster
talosctl bootstrap --nodes 10.100.50.11
talosctl config endpoints 10.100.50.10 # After bootstrap, point to VIP
talosctl kubeconfig kubeconfig.yaml --nodes 10.100.50.11

# Check health:
talosctl --nodes 10.100.50.11 health
```

## Flux bootstrap

- Activate direnv
- Add sops secret to the cluster
```sh
cat <location-of-age-key/keys.txt> | \
  kubectl create secret generic sops-age \
  --namespace flux-system \
  --from-file=age.agekey=/dev/stdin
```
- Install flux
  - Need personal access token from github, copy it
  - Export token as GITHUB_TOKEN and username as GITHUB_USER
  - flux check --pre
  - Below command:
    ```bash
    flux bootstrap github \
      --owner=atred \
      --repository=homelab \
      --branch=master \
      --path=./clusters/staging \
      --personal
    ```

## Secrets management

Whenever you create a new secret, encrypt it like so:
