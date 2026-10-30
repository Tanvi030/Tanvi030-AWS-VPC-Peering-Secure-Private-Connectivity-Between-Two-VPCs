
# AWS VPC Peering – Secure Private Connectivity Between Two VPCs

![AWS](https://img.shields.io/badge/Cloud-AWS-orange)
![Service](https://img.shields.io/badge/Service-VPC%20Peering-blue)
![Region](https://img.shields.io/badge/Region-ap--south--1-green)
![Status](https://img.shields.io/badge/Project-Implementation%20Completed-success)

## Project Information

| Detail | Information |
|---|---|
| Student Name | Tanvi Kishor Gangurde |
| Batch | October 2026 |
| Project | AWS VPC Peering |
| Cloud Provider | Amazon Web Services |
| Region | Asia Pacific (Mumbai) – ap-south-1 |
| Project Type | AWS Networking / Cloud Infrastructure |

## 1. Project Overview

This project demonstrates how to establish private network connectivity between two Amazon Virtual Private Clouds (VPCs) using VPC Peering.

Two VPCs were created with non-overlapping CIDR blocks. Three Amazon EC2 instances were deployed, including one public Bastion Host and two private application servers.

The project demonstrates secure communication between private EC2 instances using private IPv4 addresses, route tables, security groups and SSH access through a Bastion Host.

## 2. Project Objectives

- Create two independent Amazon VPCs.
- Configure public and private subnets.
- Deploy three EC2 instances.
- Establish a VPC Peering connection.
- Configure forward and return routes.
- Configure security groups for controlled traffic.
- Access private instances through a Bastion Host.
- Test connectivity in both directions.
- Demonstrate controlled network failure and recovery.

## 3. AWS Services Used

| AWS Service | Purpose |
|---|---|
| Amazon VPC | Create isolated virtual networks |
| VPC Peering | Connect two VPCs privately |
| Amazon EC2 | Host Bastion and application servers |
| Route Tables | Control network traffic paths |
| Internet Gateway | Provide internet access to Bastion |
| Security Groups | Control inbound and outbound traffic |
| Amazon Linux 2023 | Operating system for EC2 instances |
| SSH | Secure remote administration |

## 4. Network Architecture

### VPC A – 10.10.0.0/16

VPC ID: `vpc-03a81927c998d1a48`

| Resource | CIDR / Private IP |
|---|---|
| Public Subnet | 10.10.1.0/24 |
| Private Subnet | 10.10.2.0/24 |
| Bastion Host | 10.10.1.165 |
| App A | 10.10.2.49 |

### VPC B – 10.20.0.0/16

VPC ID: `vpc-0339664bf7e482224`

| Resource | CIDR / Private IP |
|---|---|
| Private Subnet | 10.20.1.0/24 |
| App B | 10.20.1.238 |

App A and App B have no public IPv4 addresses.

## 5. EC2 Instance Details

| Instance | Instance ID | Private IP |
|---|---|---|
| ewb-bastion | i-0b954854f94d87957 | 10.10.1.165 |
| ewb-app-a | i-04982c26196ecce20 | 10.10.2.49 |
| ewb-app-b | i-0298b4ab011709f90 | 10.20.1.238 |

Instance configuration:
- Operating System: Amazon Linux 2023
- Instance Type: t3.micro
- Storage: 8 GiB
- Region: ap-south-1

## 6. VPC Peering Configuration

Peering Connection Name: `ewb-a-b-peer`

Peering Connection ID:

`pcx-087d23be542b209ff`

Status: Active

Requester: VPC A  
Accepter: VPC B

Both VPCs use non-overlapping CIDR blocks, allowing the peering connection to be established.

## 7. Route Table Configuration

Three custom route tables were configured.

### ewb-rt-a-public

Associated subnet: ewb-a-public

| Destination | Target |
|---|---|
| 10.10.0.0/16 | local |
| 0.0.0.0/0 | Internet Gateway |
| 10.20.0.0/16 | VPC Peering |

### ewb-rt-a-private

Associated subnet: ewb-a-private

| Destination | Target |
|---|---|
| 10.10.0.0/16 | local |
| 10.20.0.0/16 | VPC Peering |

### ewb-rt-b-private

Associated subnet: ewb-b-private

| Destination | Target |
|---|---|
| 10.20.0.0/16 | local |
| 10.10.0.0/16 | VPC Peering |

The route tables provide forward and return traffic paths between the two VPCs.

## 8. Security Group Configuration

### ewb-sg-bastion
- SSH TCP 22 allowed from the administrator's public IP (/32).
- Default outbound traffic allowed.

### ewb-sg-app-a
- ICMP traffic allowed from VPC B.
- SSH access allowed from Bastion.
- ICMP access allowed from Bastion.

### ewb-sg-app-b
- SSH TCP 22 allowed from the Bastion subnet.
- SSH TCP 22 allowed from the App A subnet.
- All ICMP IPv4 allowed from App A private IP `10.10.2.49/32`.

## 9. Connectivity Testing

The following tests were performed.

| Test | Result |
|---|---|
| Laptop to Bastion SSH | Successful |
| Bastion to App A ping | Successful |
| Laptop to App A through Bastion | Successful |
| App A to App B ping | Successful |
| App B to App A ping | Successful |
| App A to App B SSH | Successful |

### App A to App B

Command:

`ping -c 4 10.20.1.238`

Result:
- 4 packets transmitted
- 4 packets received
- 0% packet loss

### App B to App A

Command:

`ping -c 4 10.10.2.49`

Result:
- 4 packets transmitted
- 4 packets received
- 0% packet loss

## 10. Controlled Failure and Recovery

A controlled security-group test was performed to understand how inbound rules affect connectivity.

### Test 1 – ICMP Rule Enabled

App B allowed ICMP traffic from App A.

Result: Successful ping communication.

### Test 2 – ICMP Rule Removed

The All ICMP IPv4 inbound rule was temporarily removed from the App B security group.

Command executed from App A:

`ping -c 4 10.20.1.238`

Result:
- 4 packets transmitted
- 0 packets received
- 100% packet loss

### Test 3 – ICMP Rule Restored

The ICMP inbound rule was restored with source:

`10.10.2.49/32`

The ping test was performed again.

Result:
- 4 packets transmitted
- 4 packets received
- 0% packet loss

### Conclusion

This experiment demonstrated that VPC Peering and routing alone do not guarantee connectivity. Security group rules must also permit the required traffic.

## 11. Network Traffic Flow

### Forward Path

1. App A sends traffic to App B's private IP.
2. The App A subnet route table identifies the destination VPC B CIDR.
3. Traffic is forwarded through the VPC Peering connection.
4. App B receives the traffic if its security group permits it.

### Return Path

1. App B sends a response to App A.
2. VPC B's route table identifies the VPC A CIDR.
3. Traffic returns through the same VPC Peering connection.
4. App A receives the response.

## 12. Important AWS Networking Concepts Learned

- VPC and subnet architecture.
- IPv4 CIDR addressing.
- Public and private subnet configuration.
- Route table associations.
- VPC Peering and private connectivity.
- Security group configuration.
- Bastion Host administration.
- SSH connectivity.
- ICMP troubleshooting.
- Forward and return routing.
- Network failure diagnosis.

## 13. Pending Evidence and Verification

The following items must be completed before final submission:

- Capture AWS Console screenshots of VPCs, subnets and route tables.
- Capture the Active VPC Peering connection.
- Capture inbound rules for all three security groups.
- Capture EC2 instance details.
- Confirm laptop-to-App-B SSH through Bastion.
- Record remaining subnet IDs, route table IDs, Internet Gateway ID and security group IDs.
- The baseline ping failure before peering was not captured during the build.
- Complete AWS resource cleanup and capture evidence.
- Verify remaining resources and billing.

These items are intentionally marked pending rather than reported as completed.

## 14. Project Cleanup

After saving all project evidence:

1. Terminate the three EC2 instances.
2. Verify EBS volumes and snapshots.
3. Release Elastic IPs if created.
4. Delete the lab key pair.
5. Remove VPC Peering routes.
6. Delete the VPC Peering connection.
7. Delete lab security groups after network interfaces disappear.
8. Delete subnets and custom route tables.
9. Detach and delete the Internet Gateway.
10. Delete both VPCs.
11. Check AWS Billing for remaining charges.

## 15. Conclusion

This project provided practical experience in designing and implementing private connectivity between two Amazon VPCs using VPC Peering.

It demonstrated how route tables, security groups and private IP addressing work together to enable secure communication between application servers.

The controlled failure and recovery experiment provided hands-on experience in identifying and troubleshooting network connectivity issues.

**Project:** AWS VPC Peering  
**Student:** Tanvi Kishor Gangurde  
**Batch:** October 2026  
**Region:** Mumbai – ap-south-1
