VPC Peering + EC2 Connectivity (AWS Console Walkthrough)

This project demonstrates VPC Peering between two separate VPCs in AWS (same region), and verifies private network connectivity between EC2 instances across those VPCs using ICMP ping (and optional simple HTTP pages to visually confirm which instance you’re hitting).

Why this matters (what a recruiter should take away)

In real environments, teams often split infrastructure into multiple VPCs (by app, environment, business unit, or compliance boundary). VPC Peering is one clean way to allow private traffic between VPCs without exposing services to the public internet.

Key value:

Connect services across VPCs using private IPs

Avoid routing sensitive traffic over the public internet

Common in multi-team AWS orgs: shared services VPC, app VPCs, data VPCs, etc.

Typical uses

App VPC ↔ Database VPC communication

Shared tooling VPC (logging/monitoring/bastion) ↔ workload VPCs

Migration phases: old VPC ↔ new VPC during cutover

What this project builds

Two VPCs with non-overlapping CIDRs

Example shown: 10.112.0.0/16 and 10.113.0.0/16

One EC2 instance in each VPC

A VPC Peering Connection between the VPCs

Route table updates on both sides so each VPC knows how to reach the other

Security Group rules to allow ICMP (ping)

Validation with ping (private IP → private IP)

Step-by-step (AWS Console)
1) Confirm you have 2 VPCs and 2 EC2 instances (one per VPC)

This project assumes:

Each EC2 is launched into the correct VPC/subnet

You can connect to each instance (EC2 Instance Connect or SSH)

Screenshot (project context / console view):


2) Create the VPC Peering Connection

In the AWS Console:

Go to VPC → Peering connections

Click Create peering connection

Choose:

Requester VPC: VPC A (example: 10.113.0.0/16)

Accepter VPC: VPC B (example: 10.112.0.0/16)

Create the peering connection

Screenshot (peering created / established):


If you’re peering across accounts, you must accept the peering request in the accepter account. (In this project, both are in the same account.)

Screenshot (peering confirmed/active):


3) Update Route Tables (BOTH sides)

This is the most important part people miss.

VPC A Route Table (example: 10.113.0.0/16)

Add a route so VPC A knows how to reach VPC B:

Destination: 10.112.0.0/16

Target: the VPC peering connection (pcx-…)

VPC B Route Table (example: 10.112.0.0/16)

Add a route so VPC B knows how to reach VPC A:

Destination: 10.113.0.0/16

Target: the VPC peering connection (pcx-…)

Screenshot (route table updated on one side):


Screenshot (route table updated on the other side):


✅ At the end, each route table should have:

local route for its own CIDR (automatic)

0.0.0.0/0 → IGW (if public subnet)

peering route → remote VPC CIDR via pcx-…

4) Update Security Groups (allow ICMP/ping)

To test with ping, the target instance must allow ICMP inbound from the other VPC CIDR (or from the specific peer instance security group).

Recommended (simple for demo):

Inbound rule on each instance SG:

Type: All ICMP - IPv4 (or Echo Request)

Source: peer VPC CIDR

If instance is in 10.112.0.0/16, allow ICMP from 10.113.0.0/16

If instance is in 10.113.0.0/16, allow ICMP from 10.112.0.0/16

Also confirm:

Outbound allows traffic (default SG outbound “All traffic” is fine for demo)

NACLs aren’t blocking ICMP (most default NACLs allow all)

5) Connect to EC2 and test private connectivity
Find the private IP of the peer instance

In EC2 console, select the instance and copy its Private IPv4 address.

Then, from instance A, run:

ping <peer-private-ip>


If you want the ping to stop automatically after a few packets:

ping -c 3 <peer-private-ip>


To stop a running ping manually: press Ctrl + C

Screenshot (ping test #1):


Screenshot (ping test #1a):


Screenshot (ping success across VPCs):


6) Optional: Simple web page proof (visual verification)

To make the demo more recruiter-friendly, you can serve a basic HTML page on each EC2 and show that:

Each instance responds with its own “identity” page

(Optional) You can curl the other instance’s private IP if you open port 80 internally

Example web headers shown in browser:

Screenshot (Instance page 1):


Troubleshooting checklist (fast)

If ping fails, check these in order:

Peering status = Active

Routes exist on BOTH route tables

VPC A → remote CIDR via pcx-…

VPC B → remote CIDR via pcx-…

Security Groups

Target instance allows ICMP from peer CIDR (or SG)

NACLs allow ICMP

CIDRs do not overlap

You’re pinging the peer private IP (not a placeholder like 10.113.x.x)
