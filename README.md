# Kubernetes-Cluster-Setup-Automation

Automating K3s cluster setup with Ansible

## Pre-requisites

- All nodes in the cluster should have Linux based OS installed
- All nodes in the cluster should have static IPs assigned to them
- SSH should be enabled on all nodes in the cluster
- All nodes in the cluster should have different hostnames but should use the same username and password
- The local mahine that will SSH to the cluster should have Ansible installed

## Configuration on your local machine

- Ensure the hostnames and the static IPs of the cluster nodes are added to the **/etc/hosts** file.

```shell
sudo nano /etc/hosts

# Cluster nodes
192.168.18.35 k3s-master
192.168.18.52 worker01
192.168.18.53 worker02
```

- Ensure your machine can talk securly with the cluster, which means:

1.  generate SSH key pair, copy the public key to all nodes in the cluster:

```shell
ssh-keygen
ssh-copy-id -i myKey.pub username@k3s-master
ssh-copy-id -i myKey.pub username@worker01
ssh-copy-id -i myKey.pub username@worker02
```

2. Add the hosts with the username and the location to the private key in the .ssh/config file:

```shell
  Host k3s-master
    User username
    IdentityFile ~/.ssh/myKey
  Host worker01
    User username
    IdentityFile ~/.ssh/myKey
  Host worker01
    User username
    IdentityFile ~/.ssh/myKey
```

and make sure the config file has the right permissions:

```
chmod 600 ~/.ssh/config
```

3. Test you can connect securely without providing passowrd:

```
ssh username@k3s-master
```

## Clone this repository and make some changes:

- Clone the repository
- Add the master node hostname under **[masternode]** and the worker node hostnames under **[workers]** in the **hosts** file
- Test:

```shell
ansible -i ./hosts -m ping all
```

## Ansible Playbook:

- Find the path of the configuration file in the cluster's system boot.  
   it's usually either _/boot/cmdline.txt_ or _/boot/firmware/cmdline.txt_
- Change the path used in the **install_k3s.yaml** file with the correct configuration path.

- Some tasks in the Ansible playbook require the **sudo** password to run:  
  Here are some alternatives:

### Alternative 1 - Ask Sudo password in Ansible:

```shell
  ansible-playbook -i ./hosts install_k3s.yaml --ask-become-pass
```

### Alternative 2 - Set Ansible Sudo Variable:

```shell
  ansible-playbook -i ./hosts install_k3s.yaml -e "ansible_become_password=TypeTheSudoPasswordHere"
```

### Alternative 3 - Store the Sudo Password in Ansible Vault:

1. Create a file with Ansible Vault (let's say its called password.yml) which will hold the password:

```shell
ansible-vault create password.yml
```

You need to provide a password for this file - this is not the sudo password, this is just an encryption password.  
Once you provide an encryption password, it will open the file where you can enter any sensitive data, add this line:

```
ansible_become_password: TypeTheSudoPasswordHere
```

Ansible vault will encrypt this data for you, which will be safe to include it in your source code.

2. Create a file (let's say its called vault.txt) which will hold your encryption password:

```
echo "the_encryption_password_you_used_earlier" > vault.txt
```

3. Ensure permissions on vault.txt are such that no one else can access it and do not add this file to a source control.

```
chmod 600 vault.txt
echo "vault.txt" >> .gitignore
```

4. Run the playbook:

```
ansible-playbook -i ./hosts install_k3s.yaml -e '@password.yml' --vault-password-file=vault.txt
```
