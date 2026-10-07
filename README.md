# VPN Server Ansible Setup

An Ansible playbook for deploying and configuring a VPN server with:

- [AmneziaWG](https://amnezia.org/)
- [sing-box](https://sing-box.sagernet.org/) (VLESS Reality)
- Unbound (default DNS resolver)
- UFW firewall
- SSH hardening
- Kernel/network tuning

## Requirements

On your local machine:

- Ansible
- SSH client
- Python 3

Install the required Ansible collections (`community.general` and
`ansible.posix`) if they are not already installed.

```sh
ansible-galaxy collection install community.general ansible.posix
```

The playbook currently targets Debian on `x86_64` with
systemd.

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
inventory/group_vars/all.yml
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
inventory/group_vars/
└── all.yml
```

Server keys are generated automatically on first deployment. To use fixed
keys, set `amneziawg_private_key` for AmneziaWG, and set both
`singbox_reality_private_key` and `singbox_reality_public_key` for sing-box
Reality in `inventory/group_vars/all.yml`. Leave these variables undefined to
retain an existing keypair or generate one when no key files exist.

sing-box configuration is loaded from `/etc/sing-box/` as numbered JSON
fragments. The role manages the default log, DNS, outbound, and route fragments,
and renders `30-inbound-vless-reality.json` from inventory variables. To add
your own fragment, keep its source in the repository and add it to
`singbox_custom_config_files`, for example:

```yaml
singbox_custom_config_files:
  - src: "{{ playbook_dir }}/singbox-custom/32-inbound-hysteria2.json"
    dest: "32-inbound-hysteria2.json"
```

This keeps custom fragments reproducible and validates the merged config on
each apply. Files copied directly to the server are also loaded and are left
alone by Ansible, but managing them through inventory is easier to reproduce.

The optional Tor inbound and outbound fragments are enabled by setting
`singbox_tor_enabled: true` in `inventory/group_vars/all.yml`. This also
installs the `tor` package. Leaving it false (the default) removes those three
fragments from `/etc/sing-box/`.

## Services

The playbook configures:

### AmneziaWG

* Creates server keys
* Configures the VPN interface
* Enables the interface at boot
* Adds only the configured server-side `s1`, `s2`, and `h1`–`h4` obfuscation
  values; omit entries to leave them out of the interface config

### sing-box

* Configures VLESS Reality
* Generates/uses Reality keys
* Provides proxy access

### DNS

* Selects Unbound by default for both VPNs.
* Choose `unbound`, `dnscrypt-proxy`, or `dnsproxy` with `dns_provider` in
  `inventory/group_vars/all.yml`. Each resolver listens on localhost for
  sing-box and on the configured AmneziaWG addresses for VPN clients. The
  default upstream behavior is recursive Unbound, Mullvad DoH for
  dnscrypt-proxy, or Quad9 DoQ for dnsproxy. Only the selected provider role
  runs and installs its provider.

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
