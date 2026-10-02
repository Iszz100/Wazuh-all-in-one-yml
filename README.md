# Wazuh All-in-One Native Installer for openSUSE Leap 16

Native Wazuh All-in-One installer for **openSUSE Leap 16.0 x86_64**.

This project installs the Wazuh central components directly on the host using RPM packages and systemd.

**No Docker, Podman, Wine, or Ansible is required for the final installation.**

The installer acts as a compatibility wrapper around the official Wazuh installation assistant and handles several openSUSE-specific differences automatically.

## Tested Environment

The installer has been successfully tested with:

| Component | Tested value |
|---|---|
| Operating System | openSUSE Leap 16.0 |
| Architecture | x86_64 / amd64 |
| Wazuh | 4.14.8 |
| Deployment | Single-node All-in-One |
| Service Manager | systemd |
| Container Runtime | Not required |
| Dashboard | HTTPS / TCP 443 |
| Indexer | TCP 9200 |
| Agent Events | TCP 1514 |
| Agent Enrollment | TCP 1515 |
| Wazuh API | TCP 55000 |

The tested installation completed successfully with all central services running:

```text
wazuh-indexer    active / enabled
wazuh-manager    active / enabled
filebeat         active / enabled
wazuh-dashboard  active / enabled
```

The Wazuh Indexer cluster was also verified in `GREEN` state.

## Important Notice

openSUSE Leap is not one of the primary operating systems used by the official Wazuh All-in-One installation assistant for central-component quickstart deployments.

This repository therefore provides an **openSUSE compatibility layer** around the official Wazuh installer.

It has been tested successfully on openSUSE Leap 16.0, but differences in repositories, package versions, networking, firewall configuration, available resources, or future Wazuh releases may require additional adjustment.

For production environments, test the installer on the target environment before deployment.

## Components Installed

The installer deploys:

- Wazuh Indexer
- Wazuh Manager
- Filebeat
- Wazuh Dashboard

All components run natively under systemd.

## openSUSE Compatibility Handling

The installer automatically handles several differences between openSUSE and the RPM-based distributions expected by the official Wazuh installation assistant.

These include compatibility handling for:

```text
libcap
procps-ng
gnupg2
```

The equivalent openSUSE packages provide the actual functionality while lightweight RPM metadata compatibility packages satisfy dependency-name checks used by the Wazuh installer.

The installer also handles the absence of:

```text
/usr/lib/systemd/systemd-sysv-install
```

on openSUSE Leap 16.

After installation, all Wazuh services are explicitly checked and configured to start automatically after reboot.

## Requirements

Recommended for a small Wazuh deployment:

```text
CPU     : 4 vCPU or more
RAM     : 8 GiB or more
Storage : 50 GB or more
```

A smaller lab environment may work, but resource usage should be monitored carefully.

The tested lab successfully installed Wazuh on a system with approximately:

```text
CPU       : 2 vCPU
RAM       : ~7.8 GiB
Disk free : ~30 GB before Wazuh native installation
```

This lower specification should be considered a lab configuration rather than a production recommendation.

## Fresh Installation

Clone the repository:

```bash
git clone https://github.com/Iszz100/Wazuh-all-in-one-yml.git
cd Wazuh-all-in-one-yml
```

Make the installer executable:

```bash
chmod +x install_wazuh_all_in_one_opensuse.sh
```

Optional syntax check:

```bash
bash -n install_wazuh_all_in_one_opensuse.sh
```

Run the installer:

```bash
sudo ./install_wazuh_all_in_one_opensuse.sh
```

The installer will perform preflight checks, install required openSUSE dependencies, configure compatibility packages, configure required kernel settings, invoke the official Wazuh installation assistant, and validate the final installation.

## Reinstall / Replace Existing Wazuh

If Wazuh is already installed and you intentionally want to replace it:

```bash
sudo ./install_wazuh_all_in_one_opensuse.sh --force-reinstall
```

**Warning:** this mode is destructive to the previous Wazuh installation.

The installer can detect an existing native Wazuh deployment and Wazuh Docker containers.

When `--force-reinstall` is explicitly used, existing Wazuh components may be stopped or removed before the native installation proceeds.

Do not use this option if you need to preserve the existing Wazuh configuration or data.

## Previous Wazuh Docker Installation

A previous Docker-based Wazuh deployment can occupy ports required by the native installation, including:

```text
443
9200
1514
1515
55000
```

The tested migration encountered Docker containers such as:

```text
wazuh/wazuh-dashboard
wazuh/wazuh-indexer
wazuh/wazuh-manager
```

A native Wazuh installation cannot bind those ports while the Docker deployment is still publishing them.

Use `--force-reinstall` only when replacing the previous Wazuh environment is intentional.

The installer is designed to identify Wazuh-related containers rather than indiscriminately removing unrelated Docker workloads.

## Optional Parameters

Display installer help:

```bash
sudo ./install_wazuh_all_in_one_opensuse.sh --help
```

Supported wrapper options may include:

```text
--port PORT
--open-api
--no-firewall
--ignore-hardware
--force-reinstall
```

### Custom Dashboard Port

Example:

```bash
sudo ./install_wazuh_all_in_one_opensuse.sh --port 8443
```

### Open Wazuh API Port

By default TCP 55000 is not opened automatically by the wrapper firewall configuration.

If remote API access is intentionally required:

```bash
sudo ./install_wazuh_all_in_one_opensuse.sh --open-api
```

Only expose the Wazuh API to trusted networks.

### Disable Firewall Changes

```bash
sudo ./install_wazuh_all_in_one_opensuse.sh --no-firewall
```

Use this only when firewall policy is managed separately.

## Default Network Ports

Normal All-in-One usage requires:

| Port | Protocol | Purpose |
|---|---|---|
| 443 | TCP | Wazuh Dashboard HTTPS |
| 1514 | TCP | Wazuh agent events |
| 1515 | TCP | Wazuh agent enrollment |
| 9200 | TCP | Wazuh Indexer |
| 55000 | TCP | Wazuh API |

The Wazuh API does not need to be exposed publicly for normal dashboard operation.

## Accessing the Dashboard

After installation the installer prints the dashboard address and generated credentials.

Example:

```text
Dashboard : https://192.168.1.24:443
Username  : admin
Password  : <generated-password>
```

Open:

```text
https://SERVER-IP/
```

or:

```text
https://SERVER-IP:443/
```

The certificate generated during installation may initially produce a browser certificate warning because it is not signed by a public CA.

## Changing Wi-Fi or Network

Changing Wi-Fi does not reinstall or damage Wazuh.

However, if the server receives its address using DHCP, its LAN IP address may change.

Check the current address with:

```bash
hostname -I
```

or:

```bash
ip -4 -br addr
```

If the server changes from:

```text
192.168.1.24
```

to:

```text
192.168.100.25
```

the new dashboard URL becomes:

```text
https://192.168.100.25
```

Wazuh agents configured with the old manager IP will also need a reachable manager address.

For a permanent deployment, use one of the following:

- DHCP reservation
- Static IP
- Stable DNS hostname

## Verify Installation

Check installed packages:

```bash
rpm -qa | grep -Ei '^wazuh|^filebeat'
```

Check service state:

```bash
sudo systemctl is-active \
  wazuh-indexer \
  wazuh-manager \
  filebeat \
  wazuh-dashboard
```

Expected result:

```text
active
active
active
active
```

Check boot persistence:

```bash
sudo systemctl is-enabled \
  wazuh-indexer \
  wazuh-manager \
  filebeat \
  wazuh-dashboard
```

Expected result:

```text
enabled
enabled
enabled
enabled
```

Check listening ports:

```bash
sudo ss -lntup | grep -E ':(443|9200|1514|1515|55000)\b'
```

## Reboot Test

A successful installation should survive a system reboot.

Reboot:

```bash
sudo reboot
```

After reconnecting:

```bash
sudo systemctl is-active \
  wazuh-indexer \
  wazuh-manager \
  filebeat \
  wazuh-dashboard
```

All four services should report:

```text
active
```

The tested openSUSE Leap 16 installation successfully passed this reboot test.

## Logs

Compatibility-wrapper log:

```text
/var/log/wazuh-opensuse-all-in-one.log
```

Official Wazuh installation log:

```text
/var/log/wazuh-install.log
```

View the latest Wazuh installer messages:

```bash
sudo tail -n 100 /var/log/wazuh-install.log
```

Follow installation progress:

```bash
sudo tail -f /var/log/wazuh-install.log
```

## Troubleshooting

### Port Already in Use

Check:

```bash
sudo ss -lntup | grep -E ':(443|9200|1514|1515|55000)\b'
```

If `docker-proxy` appears, check existing containers:

```bash
docker ps -a
```

Do not terminate or delete unrelated containers simply because they occupy a port. Identify the owning application first.

### Check Wazuh Services

```bash
sudo systemctl status \
  wazuh-indexer \
  wazuh-manager \
  filebeat \
  wazuh-dashboard \
  --no-pager
```

### Check Indexer

```bash
curl -k https://127.0.0.1:9200
```

An HTTP `401` response without credentials is expected and confirms that the HTTPS endpoint is responding.

### Check Dashboard

```bash
curl -k -I https://127.0.0.1:443
```

## Security Notes

Generated Wazuh credentials are sensitive.

After installation:

- Store the administrator password securely.
- Change exposed credentials when appropriate.
- Do not commit passwords or generated certificate archives to Git.
- Do not expose TCP 9200 or TCP 55000 directly to untrusted networks.
- Restrict Dashboard access according to your environment.
- Keep openSUSE and Wazuh packages updated.

## Project Scope

This project focuses specifically on providing a practical native Wazuh All-in-One deployment path for **openSUSE Leap 16**.

It does not modify the Wazuh source code.

Wazuh packages, the official installation assistant, and Wazuh itself remain projects of Wazuh, Inc.
