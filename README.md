🔗 VPC Peering + EC2 Connectivity
(AWS Console Walkthrough)

This project demonstrates VPC Peering between two separate VPCs in AWS (same region) and verifies private network connectivity between EC2 instances across those VPCs using ICMP ping.
(Optional simple HTTP pages are used to visually confirm which instance is being reached.)

🎯 Why This Matters
(What a Recruiter Should Take Away)

In real environments, teams often split infrastructure into multiple VPCs—by application, environment, business unit, or compliance boundary.

VPC Peering is one clean way to allow private traffic between VPCs without exposing services to the public internet.

Key Value

Connect services across VPCs using private IPs

Avoid routing sensitive traffic over the public internet

Common in multi-team AWS organizations:

Shared services VPC

Application VPCs

Data / analytics VPCs

🧩 Typical Real-World Uses

App VPC ↔ Database VPC communication

Shared tooling VPC (logging / monitoring / bastion) ↔ workload VPCs

Migration phases: old VPC ↔ new VPC during cutover

🏗️ What This Project Builds
Network Foundation

Two VPCs with non-overlapping CIDRs

Example shown:

10.112.0.0/16

10.113.0.0/16

Compute

One EC2 instance in each VPC

Connectivity

VPC Peering Connection between both VPCs

Route table updates on both sides so each VPC knows how to reach the other

Security Group rules to allow ICMP (ping)

Validation

Connectivity verified using private IP ping
(private IP → private IP)

✅ Final Outcome

VPC Peering connection established

Route tables updated on both sides

Security Groups allow ICMP

Private network connectivity confirmed via ping





<img width="1891" height="933" alt="vpcarmproject" src="https://github.com/user-attachments/assets/6bd50e1e-4b7c-4d70-9194-0bee235f0782" />

Step 1: EC2 Instances (One per VPC)

Each EC2 instance is launched into its respective VPC and subnet.


<img width="1915" height="958" alt="ec2peering project" src="https://github.com/user-attachments/assets/fb641e69-669c-4496-9a78-9237b74e9c91" />


Step 2: Create VPC Peering Connection

A peering connection is created between both VPCs and accepted (same AWS account).

📸 Screenshots:

First peering confirmation



<img width="1888" height="923" alt="First peering confirmation" src="https://github.com/user-attachments/assets/0b854e8b-bed4-4d80-b15d-9ed1803807d9" />



Second peering confirmation

<img width="1895" height="931" alt="Second peering confirmation" src="https://github.com/user-attachments/assets/627a5582-22e5-4e5d-818e-31f5db20b797" />

Step 3: Update Route Tables (Both Sides)

Each VPC route table includes a route to the other VPC via the peering connection.

Required routes:

In arm-project-vpc route table
10.113.0.0/16 → pcx-…

In arm project2-vpc route table
10.112.0.0/16 → pcx-…



<img width="1898" height="865" alt="Peering Proj SS" src="https://github.com/user-attachments/assets/bfbe9285-9eb3-4463-b3e6-75583edeb5cb" />

<img width="1915" height="958" alt="ec2peering project" src="https://github.com/user-attachments/assets/48b6848c-1dfe-4b43-b113-6a971f680b1c" />


Step 4: Security Groups (ICMP Allowed)

Security Groups on both EC2 instances allow ICMP from the peer VPC CIDR.

Inbound rule example:

Type: All ICMP – IPv4

Source: peer VPC CIDR (10.112.0.0/16 or 10.113.0.0/16)


screenshot for security group:

<img width="1910" height="853" alt="security group arm project" src="https://github.com/user-attachments/assets/645a4ced-41ac-4fd0-8326-da4884754754" />

Step 5: Connectivity Test (Ping)

Private IP connectivity is validated using ping between EC2 instances.

Commands used:

ping <peer-private-ip>
ping -c 3 <peer-private-ip>


To stop a running ping:

Ctrl + C


📸 Screenshots:


<img width="1762" height="938" alt="ping check 1" src="https://github.com/user-attachments/assets/c04a3ba3-398d-405b-bb4c-a89d84532738" />

<img width="1893" height="930" alt="ping check 1a" src="https://github.com/user-attachments/assets/1444a836-e2d2-4339-8f1d-9f4215f58f20" />

<img width="1918" height="915" alt="ping check 2" src="https://github.com/user-attachments/assets/f36cd7eb-cc39-4242-a2a6-cc88fb980656" />


Optional: HTTP Page Verification

Each EC2 serves a simple HTML page to visually confirm which instance is being reached.

📸 Screenshots:

<img width="1915" height="957" alt="image" src="https://github.com/user-attachments/assets/0b7a951c-bdb9-4940-b942-0e29c871bd44" />


<img width="1913" height="957" alt="image" src="https://github.com/user-attachments/assets/db7416db-cb2c-4039-bdb5-309afaab5117" />






Troubleshooting Checklist

If ping fails, verify:

Peering status = Active

Routes exist in both route tables

Security Groups allow ICMP from peer CIDR

NACLs allow ICMP

CIDRs do not overlap

You’re pinging a real private IP

NACLs allow ICMP

CIDRs do not overlap

You’re pinging the peer private IP (not a placeholder like 10.113.x.x)
