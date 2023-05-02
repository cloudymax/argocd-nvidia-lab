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

## Example VM

```bash
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: testvm
spec:
  running: false
  template:
    metadata:
      labels:
        kubevirt.io/size: small
        kubevirt.io/domain: testvm
    spec:
      domain:
        devices:
          disks:
            - name: containerdisk
              disk:
                bus: virtio
            - name: cloudinitdisk
              disk:
                bus: virtio
          interfaces:
          - name: default
            masquerade: {}
        resources:
          requests:
            memory: 64M
      networks:
      - name: default
        pod: {}
      volumes:
        - name: containerdisk
          containerDisk:
            image: quay.io/kubevirt/cirros-container-disk-demo
        - name: cloudinitdisk
          cloudInitNoCloud:
            userDataBase64: SGkuXG4=

```
