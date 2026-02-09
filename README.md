# gke-hybrid-autonomy
A hybrid Kubernetes control plane linking on-premise Incus containers with Google Cloud (GKE) via a zero-trust Tailscale mesh VPN.

🌐 The Zero-Trust Mesh Network (Tailscale)
To enable secure communication across three different environments without exposing public ports, I implemented a Zero-Trust Mesh VPN using Tailscale (WireGuard).

The Architecture
The network creates a private encrypted tunnel between my devices, allowing them to communicate as if they were on the same local switch:

GCP Gateway (USA): An e2-micro instance acting as a Subnet Router. It advertises the Google Cloud VPC range (10.128.0.0/20) to the rest of the mesh.

Mobile Terminal (S25): Acts as the mobile command center via Termux. It can securely ping and SSH into the GCP VM and the local HP Laptop from anywhere.

Local Hybrid Host (HP Ubuntu): Runs Incus containers. It is connected to the mesh to bridge local containerized GKE nodes with the Cloud control plane.

Key Configurations
IP Forwarding: Enabled on the GCP Gateway to allow traffic routing between the Tailscale interface and the VPC.

Subnet Routing: Configured --advertise-routes=10.128.0.0/20 on the cloud node so the S25 and Laptop can reach internal GCP resources without public IPs.

Persistent Connectivity: Disabled Key Expiry for the server nodes to ensure the hybrid bridge remains "immortal" and stable for long-term SRE tasks.

Verification (The SRE Proof)
From my S25 Terminal (Termux), I can verify the mesh health by reaching the internal GCP IP:

´´´bash
# Pinging the internal IP of the US-based VM from a mobile device in Germany
ping 10.128.0.2
´´´
