# VPN Server Ansible Setup

An Ansible playbook for deploying and configuring a VPN server with:

- [AmneziaWG](https://amnezia.org/)
- [sing-box](https://sing-box.sagernet.org/) (VLESS Reality)
- dnscrypt-proxy
- UFW firewall
- SSH hardening
- Kernel/network tuning

## Requirements

On your local machine:

- Ansible
- SSH client
- Python 3

Install required Ansible collections:

Clone the repository:

```fish
git clone https://git.awy.one/vpn-playbook
cd vpn-playbook
```

## Setup

Copy the example inventory:

```fish
cp inventory/hosts.ini.example inventory/hosts.ini
```

Edit `inventory/hosts.ini` and add your server details.

Example:

```ini
[vpn]
vpn01 ansible_host=<server_ip> ansible_user=root
```

Edit your variables:

```fish
group_vars/all.yml
```

Copy your public SSH key to the server.

On Linux:

```fish
ssh-copy-id -p <ssh_port> root@<server_ip>
```

Verify SSH access:

```fish
ssh -p <ssh_port> root@<server_ip>
```

Run the playbook:

```fish
ansible-playbook -i inventory/hosts.ini playbook.yml
```

## Configuration

The main configuration is stored in:

```text
group_vars/
└── all.yml
```

## Services

The playbook configures:

### AmneziaWG

* Creates server keys
* Configures the VPN interface
* Enables the interface at boot

### sing-box

* Configures VLESS Reality
* Generates/uses Reality keys
* Provides proxy access

### dnscrypt-proxy

* Uses Mullvad DNS-over-HTTPS
* Serves DNS for VPN clients internally

### Firewall

Configures UFW with:

* SSH access
* HTTP/HTTPS
* AmneziaWG UDP port
* Internal VPN DNS access

### SSH Hardening

Applies:

* Key-only authentication
* Connection limits
* Custom SSH port

## License

This project is licensed under the GNU General Public License v3.0.

See `LICENSE` for details.
