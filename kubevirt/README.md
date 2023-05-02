# Kubevirt

Kubevirt wraps QEMU and provides a Kubernetes-Native way to deploy virtual machines as code.

- Virtual Machines are defined as yaml in the `virtual-machines` directory
- Disk images are stores as PVCs and are defined in the `disks` directory

## Installing the Operator and CRDs

Following: https://kubevirt.io/quickstart_cloud/

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
