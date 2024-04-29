# Upgrade Process


```
docker build -t ghcr.io/lordmuffin/k8s-kairos:v1.28 -f Dockerfile
docker push ghcr.io/lordmuffin/k8s-kairos:v1.28
```
```
mkdir build/
docker run -v "$PWD"/build:/tmp/auroraboot -v /var/run/docker.sock:/var/run/docker.sock --rm -ti quay.io/kairos/auroraboot:v0.2.5 --set container_image=ghcr.io/lordmuffin/k8s-kairos:v1.28 --set "disable_http_server=true" --set "disable_netboot=true" --set "state_dir=/tmp/auroraboot" --debug
```

```
sudo kairos-agent upgrade --image ghcr.io/lordmuffin/custom-ubuntu-22.04-standard-amd64-generic-v2.4.3-k3sv1.28.2-k3s1:v0.0.4
```

```
cat <<EOF | sudo docker run --rm -i --net host quay.io/kairos/auroraboot \
                    --cloud-config - \
                    --set "container_image=ghcr.io/lordmuffin/custom-ubuntu-22.04-standard-amd64-generic-v2.4.3-k3sv1.28.2-k3s1:v0.0.4"
#cloud-config

debug: true
growpart:
    devices:
        - /
hostname: kairos-{{ trunc 4 .MachineID }}
install:
    auto: true
    bind_mounts:
        - /etc/kubernetes/
        - /var/lib/etcd/
        - /var/lib/kubelet/
    device: /dev/vda
    firmware: bios
    force: true
    grub-entry-name: Kairos
    no-format: false
    part-table: gpt
    partitions:
        oem:
            fs: ext4
            size: 60
        recovery:
            fs: ext4
            size: 4096
    poweroff: false
    reboot: true
reset:
    grub-entry-name: Kairos
    poweroff: true
    reboot: true
    reset-oem: false
    reset-persistent: true
strict: true
upgrade:
    grub-entry-name: Kairos
    poweroff: false
    reboot: true
    recovery: false
users:
    - lock_passwd: true
      name: kairos
      passwd: kairos
      ssh_authorized_keys:
        - ssh-rsa ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJIclbkEUSlU5EM9knO7P/t3mFCCvQUffvlQaPxxsXVG
write_files:
- content: LS0tCmFwaVNlcnZlcjoKICAgIGV4dHJhQXJnczoKICAgICAgYXV0aG9yaXphdGlvbi1tb2RlOiBOb2RlLFJCQUMKICAgIHRpbWVvdXRGb3JDb250cm9sUGxhbmU6IDRtMHMKYXBpVmVyc2lvbjoga3ViZWFkbS5rOHMuaW8vdjFiZXRhMwpjZXJ0aWZpY2F0ZXNEaXI6IC9ldGMva3ViZXJuZXRlcy9wa2kKY2x1c3Rlck5hbWU6IGs4cy11YnVudHUKY29udHJvbGxlck1hbmFnZXI6IHt9CmRuczoge30KZXRjZDoKICBsb2NhbDoKICAgIGRhdGFEaXI6IC92YXIvbGliL2V0Y2QKaW1hZ2VSZXBvc2l0b3J5OiByZWdpc3RyeS5rOHMuaW8Ka2luZDogQ2x1c3RlckNvbmZpZ3VyYXRpb24KbmV0d29ya2luZzoKICAgIGRuc0RvbWFpbjogaG9tZS5sYWIKICAgIHBvZFN1Ym5ldDogMTAuMjQ0LjAuMC8xNgogICAgc2VydmljZVN1Ym5ldDogMTAuOTYuMC4wLzE4CnNjaGVkdWxlcjoge30KLS0tCmFwaVZlcnNpb246IGt1YmVsZXQuY29uZmlnLms4cy5pby92MWJldGExCmtpbmQ6IEt1YmVsZXRDb25maWd1cmF0aW9uCnNlcnZlclRMU0Jvb3RzdHJhcDogdHJ1ZQotLS0KYXBpVmVyc2lvbjoga3ViZWFkbS5rOHMuaW8vdjFiZXRhMwpraW5kOiBJbml0Q29uZmlndXJhdGlvbgpub2RlUmVnaXN0cmF0aW9uOgogIGNyaVNvY2tldDogIi9ydW4vY29udGFpbmVyZC9jb250YWluZXJkLnNvY2siCi0tLQ==
  encoding: b64
  owner: root
  path: /etc/kubernetes/kubeadm-config.yaml
  permissions: "0644"

EOF
```