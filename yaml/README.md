# Introduction

- [Basic Tutoral of Wordpress on Kubernetes](https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/)

In case more fine grained control over what happens in the wordpress installation is required, 
we can deploy the system using our own custom deployment yaml files using the description in this
tutorial.

```bash
cat <<EOF >./kustomization.yaml
secretGenerator:
- name: mysql-pass
  literals:
  - password=YOUR_PASSWORD
EOF
```

```bash
cat <<EOF >>./kustomization.yaml
resources:
- mysql-deployment.yaml
- wordpress-deployment.yaml
EOF
```

# NFS Tutorial

Open ports 111 and 2049 for both TCP and UDP.

- [Link to tutorial](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-16-04)
- [NFS Firewall Ports](https://serverfault.com/questions/377170/which-ports-do-i-need-to-open-in-the-firewall-to-use-nfs)


rules in exports:
```
/var/nfs/general    *(rw,sync,no_subtree_check,no_root_squash)
```

get directory size:
```bash
du -hs /var/nfs/general
```

Bash Commands:
```bash
sudo apt-get update
sudo apt-get install nfs-kernel-server
sudo mkdir /var/nfs/general -p
ls -la /var/nfs/general
sudo chown nobody:nogroup /var/nfs/general
sudo chmod 777 /var/nfs/general/

sudo nano /etc/exports
# Change acording to the reule in top -> /var/nfs/general    *(rw,sync,no_subtree_check,no_root_squash)

sudo systemctl restart nfs-kernel-server

# add firewall rules!
```
