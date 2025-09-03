# Linux Troubleshooting Guide

A practical, no-fluff collection of real-world Linux issues and how they were fixed. Each entry shares the symptoms, what caused the problem, and the step-by-step fix—so the same issue doesn’t waste time twice. Focus areas include system boot, networking, storage, package management, drivers, services, and performance. Contributions and pull requests are welcome.

---

## Fixing KVM Bridge/NAT Internet Access Blocked by firewalld

When using a KVM virtual machine with a bridged network, like the default NAT network virbr0, the host's firewall, firewalld, often blocks the VM's internet access. This happens because firewalld is not configured by default to allow traffic to be forwarded between your physical network and the virtual bridge.

This guide provides the commands to fix this by adding the necessary forwarding rules to firewalld.

---

## The Solution

You need to add permanent, direct passthrough rules to your firewall configuration to explicitly allow traffic to and from the KVM bridge.

### Step 1: Add Firewall Rules

Run the following commands in your host's terminal. These rules instruct firewalld to accept traffic being forwarded in both directions on the virbr0 interface.

`sudo firewall-cmd --permanent --direct --passthrough ipv4 -I FORWARD -i virbr0 -j ACCEPT`

`sudo firewall-cmd --permanent --direct --passthrough ipv4 -I FORWARD -o virbr0 -j ACCEPT`

Note: If you are using a custom bridge, for example br0, replace virbr0 with the name of your bridge in both commands.

### Step 2: Reload the Firewall

For the new permanent rules to take effect, you must restart or reload the firewalld service.

`sudo systemctl restart firewalld`

Your KVM virtual machine should now have internet access.

---

## Command Breakdown

* **--permanent**: Makes the rule persistent across reboots.
* **--direct --passthrough ipv4**: Allows you to add a custom rule directly into the iptables chains, in this case the FORWARD chain for IPv4.
* **-I FORWARD**: Inserts the rule into the FORWARD chain, which handles packets being routed through the host.
* **-i virbr0**: Specifies the rule applies to traffic coming from the input interface virbr0.
* **-o virbr0**: Specifies the rule applies to traffic going to the output interface virbr0.
* **-j ACCEPT**: Sets the action for matching packets to ACCEPT, which means to allow them.
