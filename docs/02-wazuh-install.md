# 2. Wazuh Server Installation

## Base OS

Ubuntu Server 22.04.5 LTS was installed on `wazuh-server`, with OpenSSH enabled during setup. 22.04 LTS was chosen because it is an officially supported release for Wazuh; a non-LTS release (e.g. 22.10) was avoided because its package repositories go out of support quickly.

```
sudo apt update && sudo apt upgrade -y
sudo apt install curl -y
```

## All-in-one install

Wazuh 4.14 was installed using the official installation assistant, which sets up the manager, indexer and dashboard together on a single host:

```
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

This took roughly 15-20 minutes and downloaded about 1 GB. The installer:

1. Verified system requirements
2. Generated certificates for the indexer, dashboard and Filebeat
3. Installed and started the **Wazuh indexer** (stores alerts)
4. Installed and started the **Wazuh manager** (analyzes logs, generates alerts)
5. Installed **Filebeat** (ships alerts from the manager to the indexer)
6. Installed and started the **Wazuh dashboard** (web UI)
7. Printed the `admin` dashboard credentials at the end

## Securing the install

The admin password printed at install time can be replaced with a password of the operator's choosing using the password tool:

```
curl -so wazuh-passwords-tool.sh https://packages.wazuh.com/4.14/wazuh-passwords-tool.sh
sudo bash wazuh-passwords-tool.sh -u admin -p '<new-password>'
```

To prevent an accidental `apt upgrade` from pulling in a newer, untested Wazuh version, the Wazuh package repository was disabled after install:

```
sudo sed -i "s/^deb/#deb/" /etc/apt/sources.list.d/wazuh.list
sudo apt update
```

## Accessing the dashboard

With the `wazuh-dashboard` port-forward rule in place (see [01-lab-setup.md](01-lab-setup.md)), the dashboard is reached from the Windows host at:

```
https://127.0.0.1:8443
```

The browser shows a certificate warning because Wazuh generates a self-signed certificate during install; this is expected in a lab environment.

## Lesson learned

A custom rules file edited by hand produced an XML syntax error (`level-"14"` instead of `level="14"`), which stopped the Wazuh manager from starting. This was caught before causing downtime by always running:

```
sudo /var/ossec/bin/wazuh-analysisd -t
```

before restarting the manager — a syntax check against the rules files.
