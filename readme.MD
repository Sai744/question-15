
Investigate and resolve an EKS issue where pods cannot
 communicate with external services.


# Check Pod Network Connectivity
The first thing to check is whether the pods have proper networking access to external services (e.g., public APIs or services on the internet).
Issue investigated: pods cannot reach external services, such as websites or APIs, and diagnose whether the issue is network-related.
Resolved Steps: 
Run a test pod:
kubectl run -i --tty --rm debug --image=busybox --restart=Never --namespace default -- /bin/sh
Once inside the pod, try pinging an external service (e.g., Google's DNS server):
ping 8.8.8.8
Try accessing an HTTP service as well:
wget google.com
If pinging or HTTP requests succeed, the pod has external connectivity, and the issue might be related to a specific service or configuration.
If pinging or HTTP requests fail, proceed to the next steps to check networking configurations.

# Step 2: Verify VPC/Subnet Configuration
Issue Investigated:
Check the VPC and subnet configuration to ensure that the pods have the correct routes for accessing external services.
Action:
Go to VPC > Subnets in the AWS console and verify that the subnet associated with your EKS worker nodes has the necessary route to an Internet Gateway (for public subnets) or a NAT Gateway (for private subnets).
Expected Outcome:
Subnets should have routes directing outbound traffic (0.0.0.0/0) to an Internet Gateway or NAT Gateway depending on the architecture.
aws ec2 describe-route-tables --region <region-name>

# Step 3: Verify NAT Gateway (for Private Subnets)

Issue Investigated:
If EKS worker nodes are running in a private subnet, ensure that traffic can be routed to the internet via a NAT Gateway.
Action:
Verify that the NAT Gateway is running in a public subnet and is correctly configured to allow outbound traffic for instances in private subnets.
Expected Outcome:
NAT Gateway should be set up in the public subnet to route traffic for private subnets to the internet.

# Step 4: Check Security Groups

Issue Investigated:
Confirm that the security groups associated with EKS worker nodes and pods allow necessary inbound and outbound traffic to/from the internet.
Action:
Verify that the EKS worker node's security group allows outbound traffic to 0.0.0.0/0 (or a restricted range, as required).
Ensure that the security group associated with pods (if using VPC CNI) allows outbound traffic to external services.
Command:
kubectl get securitygroup <security-group-id>
Expected Outcome:
Security groups should have outbound rules allowing traffic to 0.0.0.0/0 (or to an appropriate range, depending on your network architecture).

# Step 5: Verify IAM Role for EKS Worker Nodes

Issue Investigated:
Ensure that the IAM role assigned to your EKS worker nodes has sufficient permissions for managing networking and accessing AWS resources (e.g., VPC, NAT Gateway).
Action:
Verify that the IAM role for the EKS worker nodes includes the necessary policies:
oAmazonEKSWorkerNodePolicy
oAmazonEC2ContainerRegistryReadOnly
oAmazonVPCFullAccess (for VPC CNI)
Command:
aws iam list-attached-role-policies --role-name <eks-node-role-name>
Expected Outcome:
IAM role should have the AmazonVPCFullAccess policy (or equivalent) to allow worker nodes to manage networking.

![60](https://github.com/user-attachments/assets/2ed285c7-b4a4-4c47-85aa-53347558551c)






