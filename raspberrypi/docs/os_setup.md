# Raspberry Pi Operating System Setup

I will use [HypriotOS](https://blog.hypriot.com) as it comes with the docker
packages preinstalled. This will somewhat speedup our installation and setup process.

As of [v1.9.2](https://kubernetes.io/docs/setup/independent/install-kubeadm/#installing-docker)
the highest compatible docker version is v17.03. We will be using
this [HypriotOS Images](https://github.com/hypriot/image-builder-rpi/releases/download/v1.4.0/hypriotos-rpi-v1.4.0.img.zip)
for that.

### Burn OS image to SD cards
A good tool to do that is [Etcher](https://etcher.io).

### Configure key based SSH Access
Run the following command to setup authorized_keys for the nodes.

Default login:
* Username: pirate
* Password: hypriot

```
$ ansible-playbook -i inventory.yml adhoc/upload_ssh_keys.yml -k
```

### OS related configurations (e.g. hostname, swap)

```
$ ansible-playbook -i inventory.yml os.yml
```

***NOTE***: The servers will reboot after executing the playbook.

### Install Kubernetes packages

```
$ ansible-playbook -i inventory.yml k8s.yml
```
