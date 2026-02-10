🚀 Project: gke-hybrid-autonomy
A hybrid Kubernetes control plane linking on-premise Incus containers with Google Cloud (GKE) via a zero-trust Tailscale mesh VPN.

🏗 Infrastructure Architecture
The network creates a private encrypted tunnel between environments, allowing them to communicate as if they were on the same local switch:

GCP Gateway (USA): An e2-micro instance acting as a Subnet Router. It advertises the Google Cloud VPC range (10.128.0.0/20) to the rest of the mesh.

Mobile Terminal (S25): The mobile command center via Termux. Securely pings and SSHs into GCP VMs and the local HP Laptop from anywhere.

Local Hybrid Host (HP Ubuntu): Runs Incus containers. Bridges local containerized nodes with the Cloud control plane.

🌐 The Zero-Trust Mesh (Tailscale)
To enable secure communication without exposing public ports, I implemented a Zero-Trust Mesh VPN using Tailscale (WireGuard).

Key Configurations:

IP Forwarding: Enabled on the GCP Gateway and Local Host to allow traffic routing.

Subnet Routing: Configured --advertise-routes=10.128.0.0/20 on the cloud node so the S25 and Laptop can reach internal GCP resources.

Persistent Connectivity: Disabled Key Expiry for server nodes to ensure the hybrid bridge remains stable for long-term SRE tasks.

Energy Management: Host power management disabled (systemctl mask sleep.target) to ensure 24/7 availability.

✅ Verification (The SRE Proof)
From my S25 Terminal (Termux), I can verify the mesh health by reaching the internal GCP IP directly from Germany to a US-based VM:

```Bash
# Pinging the internal IP of the US-based VM from a mobile device
ping  -c 4 10.128.0.2
# Success: 0% packet loss, RTT ~140ms (via direct Tailscale path)
📊 Monitoring & Observability
Prometheus: Scrapes metrics from the HP Laptop via node_exporter on Port 9100.
```
Incus Isolation: Monitoring runs in a dedicated sre-lab container to ensure observability even if host services are being redeployed.

🛠 Quick Start / Cheatsheet
SSH to Host: ssh hp (configured via ~/.ssh/config)

Container Ops: incus list, incus snapshot create <node> <label>

Check Mesh Status: tailscale status

🛠 Lessons Learned & Troubleshooting
During the build-out of this hybrid environment, several real-world networking hurdles were cleared. Documenting these ensures reproducible success:

1. The "Default-Deny" Trap (SSH & Node Exporter)
Issue: Even with Tailscale active, attempts to SSH or scrape metrics resulted in Connection Refused.

Root Cause: Standard Linux Desktop installations (like the HP Pavilion) do not ship with an SSH server by default. Additionally, many exporters bind to 127.0.0.1 by default for security.

Solution: * Installed openssh-server.

Modified /etc/default/prometheus-node-exporter to use --web.listen-address=:9100 to allow the Incus bridge and Tailscale interfaces to reach the service.

2. Container-to-Host Communication
Issue: Incus containers could not reach the Host's Tailscale IP initially.

Root Cause: Firewall (UFW/iptables) rules on the Host were blocking the virtual bridge interface (incusbr0).

Solution: Explicitly allowed traffic from the Incus bridge: sudo ufw allow in on incusbr0 to any port 9100.

3. Maintaining "Immortal" Hardware
Issue: The hybrid bridge would disconnect whenever the laptop lid was closed or the system entered power-save mode.

Root Cause: Systemd power management targets (suspend.target, sleep.target) were active.

Solution: Masked all sleep targets and configured systemd-logind to ignore the lid switch. This transformed a consumer gaming laptop into a reliable, "always-on" SRE node.

4. Direct Path vs. DERP Relay
Issue: Initial pings showed high latency (>300ms) despite being on the same local network.

Root Cause: Tailscale was using a DERP relay (Frankfurt) because NAT traversal hadn't established a direct peer-to-peer path yet.

Solution: Once the direct path was established via UDP hole punching, latency dropped to <50ms, verifying the efficiency of the WireGuard-based mesh.
