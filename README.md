# ansible-5-minutes-server

This ansible playbook sets up what's described in the article ["My First 5 Minutes On A Server; Or, Essential Security for Linux Servers"](https://web.archive.org/web/20201112012219/https://plusbryan.com/my-first-5-minutes-on-a-server-or-essential-security-for-linux-servers).

The ansible scripts should work with the Ubuntu linux distribution and have a hard dependency on that.

## Getting started

1. Edit personal information in `secrets.yml` with [`ansible-vault`](https://docs.ansible.com/ansible/latest/user_guide/vault.html) (default password: **password**)
   ```
   mv secrets.default secrets.yml
   ansible-vault rekey secrets.yml
   ansible-vault edit secrets.yml
   ```
2. Add your server by creating the `inventory` file with the following contents:
   ```
   [groupname]
   server.ip.or.hostname
   ```
   `[groupname]` can be anything to your liking. You can have multiple groups.
3. Run (replace `<SUDOER>` with the login name)
   ```
   ansible-playbook -u <SUDOER> --private-key=~/.ssh/id_rsa.pub --ask-become-pass --ask-vault-pass --inventory-file=inventory playbook.yml
   ```

Your server is now bootstrapped.

## Included roles

### `bootstrap`

This is the baseline for every server and the initial reason these scripts exist.

- change the root password
- add an "admin" user (username of your choice)
- add your public key to `.ssh/authorized_keys` for the admin user
- add that admin user to `/etc/sudoers`
- install [ufw](https://launchpad.net/ufw) (uncomplicated firewall, an iptables frontend) and [fail2ban](https://www.fail2ban.org/)
- SSH: disallow password authentication
- SSH: disallow root login
- setup firewall to allow SSH traffic
- install additional apt packages provided by the user
- add a "deploy" user (username of your choice) _without sudo_ usage permission
- setup periodic [unattended-upgrades](https://wiki.debian.org/UnattendedUpgrades) with automatic reboots
- setup [sSMTP](https://wiki.debian.org/sSMTP) to send (system) mails via SMTP
- create a 1GB file at `/swapfile` and swap on it
- set sysctl `vm.swappiness = 0`

This repository is based in [guillaumevincent/Ansible-My-First-5-Minutes-On-A-Server](https://github.com/guillaumevincent/Ansible-My-First-5-Minutes-On-A-Server). Furthermore it contains several changes suggested in [PR#1 of that repository](https://github.com/guillaumevincent/Ansible-My-First-5-Minutes-On-A-Server/pull/1) by [@joraman](https://github.com/joraman).

### ruby-rvm

If you choose to apply this role to a host, it will:

- set up [rvm](https://rvm.io/) via the [dedicated ubuntu deb package](https://github.com/rvm/ubuntu_rvm).
- install required dependencies to install ruby via apt.
- install the specified ruby version for the deploy user.

Example inventory:

```
[ruby]
ubuntu20 ruby_version=2.7.2
```

## Modify personal information

All your personal information is stored in an encrypted file with `ansible-vault`.

1. Move `secrets.default` to `secrets.yml`
   ```
   mv secrets.default secrets.yml
   ```
2. Change your vault password (existing password: **password**)
   ```
   ansible-vault rekey secrets.yml
   ```
   Follow the instructions to change the password.
3. Edit secrets information
   ```
   ansible-vault edit secrets.yml
   ```

## Testing locally

Install [vagrant](https://www.vagrantup.com/docs/installation).

```
vagrant up
ansible-playbook -i inventory/test --ask-vault-pass playbook.yml
```

These vagrant boxes are publicly officially available, not something custom made. The first run of `vagrant up` can take a a long time to set everything up.

This sets up a machine called `ubuntu20` using Ubuntu 20.04 "focal" (see `Vagrantfile`)

### Helpful ~/.ssh/config addition when testing

For your convenience, add this to your `~/.ssh/config`

```ssh
Host ubuntu20
  Hostname 10.0.1.2
  User vagrant
  IdentityFile ~/.ssh/id_rsa
  StrictHostKeyChecking no
```

### Bootstrap a specific vagrant box

For instance the `ubuntu20`:

```
vagrant up ubuntu20
ansible-playbook -u vagrant --private-key=~/.ssh/id_rsa.pub --ask-vault-pass -i 'ubuntu20,' playbook.yml
```

When done, destroy the machine to start with a blank slate on the next run:

```
vagrant destroy
```

## Debugging

### Can't connect to the serer

Verify that you can ping your server via SSH:

```
ansible all --user=<SUDOER> --private-key=<SSH_PRIVATE_KEY> --inventory-file='<IP>,' -m ping
```

Example with an `inventory` file:

```
ansible all --user=vagrant --private-key=~/.ssh/id_rsa --inventory-file=inventory/test -m ping
```

```
unbunt20 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

Explanations:

- `ansible all` : run ansible command on all hosts.
- `--user=vagrant` : login via ssh with user `vagrant`.
- `--private-key=~/.ssh/id_rsa` : path of the ssh key use to connect to the server.
- `--inventory-file=inventory/test` : specify inventory file.
- `-m ping` : run module ping.

If the ping run fails, your user doesn't have access to the machine.

### Ansible can't execute apt commands

To allow ansible to work on older Ubuntu versions, you might need to manually
`apt install python-minimal python-zipstream`
on the remote server.
