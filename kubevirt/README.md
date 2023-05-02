# Kubevirt

Following: https://kubevirt.io/quickstart_cloud/

## Installing the Operator and CRDs

> You can manually download assets by checking the releases links: https://github.com/kubevirt/kubevirt/releases/

- Installing the cli from source

  ```bash
  VERSION=$(kubectl get kubevirt.kubevirt.io/kubevirt -n kubevirt -o=jsonpath="{.status.observedKubeVirtVersion}")
  ARCH=$(uname -s | tr A-Z a-z)-$(uname -m | sed 's/x86_64/amd64/') || windows-amd64.exe
  echo ${ARCH}
  curl -L -o virtctl https://github.com/kubevirt/kubevirt/releases/download/${VERSION}/virtctl-${VERSION}-${ARCH}
  chmod +x virtctl
  sudo install virtctl /usr/local/bin
  ```

- Install as a krew plugin

  ```bash
  kubectl krew install virt
  ```

- Enable nested virtualization if desired

  ```bash
  kubectl -n kubevirt patch kubevirt kubevirt \
    --type=merge \
    --patch '{"spec":{"configuration":{"developerConfiguration":{"useEmulation":true}}}}'
  ```
