# Ansible: My First 5 Minutes On A Server

This ansible playbook sets up what's described in the article ["My First 5 Minutes On A Server; Or, Essential Security for Linux Servers"](https://plusbryan.com/my-first-5-minutes-on-a-server-or-essential-security-for-linux-servers).

This repository is based in [guillaumevincent/Ansible-My-First-5-Minutes-On-A-Server](https://github.com/guillaumevincent/Ansible-My-First-5-Minutes-On-A-Server). Furthermore it contains several changes suggest in [PR#1 of that repository](https://github.com/guillaumevincent/Ansible-My-First-5-Minutes-On-A-Server/pull/1) by [@joraman](https://github.com/joraman).

The ansible scripts should work with the Ubuntu linux distribution and have a hard dependency on that.

TL;DR:

- change the root password
- add an admin user (name is your choice)
- add your public key to .ssh/authorized_keys for the admin user
- add that admin user to /etc/sudoers
- install default packages (ufw, fail2ban, ...)
- install user extra packages
- setup firewall
- disallow password authentication
- disallow root SSH access

To allow ansible to work, you might need to manually
`apt install python-minimal python-zipstream`
on the remote server.

## Getting started

Edit personal information in `secrets.yml` ansible vault (default password: **password**)

```
mv secrets.default secrets.yml
ansible-vault rekey secrets.yml
ansible-vault edit secrets.yml
```

Run (replace `<SUDOER>`, `<SSH_PRIVATE_KEY>`, `<IP>`)

```
ansible-playbook -u <SUDOER> --private-key=<SSH_PRIVATE_KEY> --ask-become-pass  --ask-vault-pass -i '<IP>,' bootstrap.yml
```

## Requirements

- ansible 2+
- git

## Checks

Verify that you can ping your server via SSH:

```
ansible all --user=<SUDOER> --private-key=<SSH_PRIVATE_KEY> --inventory-file='<IP>,' -m ping
```

Example for a Raspberry Pi:

```
ansible all --user=pi --private-key=~/.ssh/id_rsa --inventory-file='192.168.1.100,' -m ping
```

```
192.168.1.100 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Explanations:

- `ansible all` : run ansible command on all hosts
- `--user=pi` : ssh with user pi
- `--private-key=~/.ssh/id_rsa` : path of the ssh key use to connect to the server
- `--inventory-file='192.168.1.100,'` : specify inventory file. do not forget the comma `,`
- `-m ping` : run module ping

If the previous command failed, `<SUDOER>` doesn't have access.

## Modify personal information

All your personal information is stored in an encrypted file with ansible-vault.

Move `secrets.default` to `secrets.yml`

```
mv secrets.default secrets.yml
```

Change your vault password (existing password: **password**)

```
ansible-vault rekey secrets.yml
```

Follow the instructions to change ansible vault password.

Edit secrets information

```
ansible-vault edit secrets.yml
```

The encrypted information that you can change:

```yml
---
ROOT_PASSWORD: "xxxxxx"
ADMIN_PASSWORD: "xxxxxx"
ADMIN_USERNAME: admin
PUBLIC_KEYS:
  - ~/.ssh/id_rsa.pub
#EXTRA_PACKAGES:
#  - vim
#  - htop
```

## Setup your server

Now run `bootstrap.yml`

```
ansible-playbook --user=<SUDOER> --private-key=<SSH_PRIVATE_KEY> --ask-become-pass  --ask-vault-pass --inventory-file='<IP>,' bootstrap.yml
```

The Raspberry Pi from above would look like this::

```
ansible-playbook --user=pi --private-key=~/.ssh/id_rsa --ask-become-pass --ask-vault-pass --inventory-file='192.168.1.100,' bootstrap.yml
```

## Testing locally

Install [vagrant](https://www.vagrantup.com/docs/installation).

```
cd test
vagrant up
cd ..
ansible-playbook -i test/test_inventory --ask-vault-pasbootstrap_roles.yml
```

These vagrant boxes are publicly officially available, not something custom made. The first run of `vagrant up` can take a a long time to set everything up.

### Test only one box

For instance the Ubuntu 20.04 "focal":

```
vagrant up ubuntu20
ansible-playbook -u vagrant --private-key=~/.ssh/id_rsa.pub --ask-become-pass  --ask-vault-pass -i 'ubuntu20,' bootstrap.yml
```

When done, destroy the machine to start with a blank slate on the next run:

```
vagrant destroy
```

### Helpful ~/.ssh/config addition

For your convenience, add this to your `~/.ssh/config`

```ssh
Host ubuntu20
  Hostname 10.0.1.2
  User vagrant
  IdentityFile ~/.ssh/id_rsa
```

## FAQ

### Can I run bootstrap.yml playbook on multiple IPs?

Change the `--inventory-file` parameter

```
ansible-playbook .... --inventory-file='<IP>,<IP2>,<IP3>,' bootstrap.yml
```

### I do not have access to my server via ssh, how do I do?

You need ssh access to use ansible. If you have the password of a sudoer, then copy your **ssh public key** on your server.

```
ssh-copy-id -i ~/.ssh/id_rsa.pub <SUDOER>@<IP>
```
