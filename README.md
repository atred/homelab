# My Kubernetes Homelab

TODO real docs

## k3s bootstrap
Bootstrap Notes:
- Setup k3s without helm controller (flux will do this later)
```sh
sudo su -
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable=helm-controller" sh
```
- Copy k3s kubeconfig to this directory
```sh
# On remote machine
sudo chmod 755 /etc/rancher/k3s/k3s.yaml
# On host machine
scp <username>@<server-address>:/etc/rancher/k3s/k3s.yaml .
mv k3s.yaml kubeconfig.yaml
# edit this to use the correct server address, not loopback
```
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

1. Download the talos iso with the following extensions:
  - siderolabs/i915
  - siderolabs/intel-ucode
  - siderolabs/iscsi-tools
2. Install it on the nodes
