# My Kubernetes Homelab

TODO real docs

Bootstrap Notes:
- Setup k3s without helm controller (flux will do this later)
- Copy k3s kubeconfig to this directory
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

Encrypting secrets:
```sh
sops --age=$AGE_PUBLIC --encrypt --encrypted-regex '^(data|stringData)$' --in-place <filename.yaml>
```


