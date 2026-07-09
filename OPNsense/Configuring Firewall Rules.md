## Configuring Firewall Rules

Go to `Firewall` > `Rules`, then select the VLAN to edit its ruleset.

![Firewall rules per VLAN](Images/Pasted%20image%2020260707195648.png)

Rules are evaluated top down and the first match wins, so order matters. Each VLAN ends in an explicit deny.

The full per-VLAN ruleset is documented in [Overview/Firewall Rules](../Overview/Firewall%20Rules.md).
