# 5. Custom Detection Rule

## Goal

The built-in rule 40112 already detects "multiple authentication failures followed by a success" at level 12 (High). The goal of the custom rule was to go one step further: treat this specific pattern as a likely **account compromise**, raise its severity, give it a clearer analyst-facing description, and map it to a MITRE ATT&CK technique.

## The rule

File: [`rules/local_rules.xml`](../rules/local_rules.xml)

```xml
<group name="local,custom,ssh,">
  <rule id="100100" level="14">
    <if_sid>40112</if_sid>
    <description>LAB: Possible compromised account - SSH brute force succeeded for $(dstuser) from $(srcip)</description>
    <mitre>
      <id>T1110</id>
    </mitre>
  </rule>
</group>
```

### How it works

- **`id="100100"`** — custom rule IDs must be 100000 or higher so they never collide with Wazuh's built-in rule set.
- **`level="14"`** — raised from the base rule's level 12, reflecting the higher confidence that this specific pattern represents a compromised credential rather than just a noisy failed-login pattern.
- **`<if_sid>40112</if_sid>`** — this rule only evaluates after rule 40112 has already matched, so it builds on (rather than duplicates) Wazuh's existing correlation logic.
- **`$(dstuser)` / `$(srcip)`** — Wazuh substitutes these with the actual targeted username and the actual source IP from the triggering event, so every alert is self-describing without needing to open the raw log.
- **`<mitre><id>T1110</id></mitre>`** — tags the alert with MITRE ATT&CK technique **T1110 (Brute Force)**, which is useful for mapping alerts to a standard framework in reporting.

## Deployment

1. Appended to `/var/ossec/etc/rules/local_rules.xml` on the Wazuh server.
2. Validated before applying:
   ```
   sudo /var/ossec/bin/wazuh-analysisd -t
   ```
   This step is important — an earlier hand-edited version of this file had a typo (`level-"14"` instead of `level="14"`) that silently prevented the Wazuh manager from starting. Running the syntax check before restarting catches this kind of mistake without causing downtime.
3. Applied:
   ```
   sudo systemctl restart wazuh-manager
   ```

## Verification

Re-running the Hydra brute-force scenario (see [04-attack-scenarios.md](04-attack-scenarios.md)) produced a new alert from rule **100100** at level 14, with the MITRE T1110 tag attached:

![Rule 100100 alert](../screenshots/04-rule-100100.png)

## Possible extensions

- A time-window version that fires on N failed logins within X seconds, independent of whether a success follows (catches attacks that don't succeed).
- An **active response** that automatically blocks the source IP via `iptables` after this rule fires.
- Applying the same pattern to other services (e.g. a web login form) once one is added to the lab.
