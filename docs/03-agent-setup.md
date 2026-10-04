# 3. Agent Deployment (ubuntu1)

## Why an agent is needed

The Wazuh server only analyzes logs it receives. Without an agent running on an endpoint, the server has no visibility into that machine, so this step connects `ubuntu1` to `wazuh-server` as a monitored host.

## Preparing the target

OpenSSH was installed on `ubuntu1` so it has a service worth attacking later, along with a throwaway account used only for the brute-force test:

```
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
sudo adduser testuser
```

## Generating the install command

From the Wazuh dashboard: **Deploy new agent** → OS: Linux (DEB amd64) → Server address: `10.0.3.8` → Agent name: `ubuntu1`. The wizard generates an install command with the correct package version for the installed server (4.14.8), which matters because an agent should not be newer than the manager it reports to.

## Installing the agent

```
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.8-1_amd64.deb
sudo WAZUH_MANAGER='10.0.3.8' WAZUH_AGENT_NAME='ubuntu1' dpkg -i ./wazuh-agent_4.14.8-1_amd64.deb
```

The `WAZUH_MANAGER` and `WAZUH_AGENT_NAME` environment variables are read by the installer and written into `/var/ossec/etc/ossec.conf`, so the agent knows where to report and how to identify itself.

```
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
sudo systemctl status wazuh-agent
```

## Verifying the connection

In the dashboard, **Agents summary** should show **Active (1)**.

![Agent active](../screenshots/05-agent-active.png)

## Lesson learned: a malformed install command

An early attempt split the install command across multiple shell arguments incorrectly, which caused the installer to write junk values (`dpkg`, `-i`, `wget`, `sudo`, etc.) into the `<address>` fields of `ossec.conf` instead of a single manager IP. The agent failed to start with a "No client configured" error.

Diagnosis:
```
sudo grep -n "address" /var/ossec/etc/ossec.conf
```

Fix: purge the broken install and reinstall with the environment variables placed correctly, as a single well-formed command, rather than trying to patch the generated config by hand:
```
sudo apt purge wazuh-agent -y
sudo rm -rf /var/ossec
```
then repeat the install step above.
