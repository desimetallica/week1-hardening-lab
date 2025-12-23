# week1-hardening-lab

## Objective

- Deploy EC2 on AWS (free tier) 
- Harden Linux (SSH, users, firewall, permissions)
- Simple bash script for config backup
- Folders: terraform, ansible

## Environment
- AWS Free Tier account
- Terraform
- Ansible
- Amazon Linux 2/2023 or Ubuntu/Debian EC2 instances

## Steps
1. Prepare your AWS account and credentials (`aws configure`).
2. Generate or use an existing SSH key pair for EC2 access.
3. Edit `terraform/defaults.tfvars` with your AMI ID, region, instance type, and SSH key paths.
4. Initialize and apply Terraform:
   ```sh
   terraform init terraform/
   terraform apply -var-file=terraform/defaults.tfvars terraform/
   ```
5. Copy the `ansible_inventory` output from Terraform to `ansible/hosts`.
6. Run the Ansible playbook:
   ```sh
   ansible-playbook -i ansible/hosts ansible/host_hardening.yml --private-key <path-to-private-key>
   ```

## Terraform Files
- `terraform/main.tf`: Main configuration (EC2, key pair, security group)
- `terraform/variables.tf`: Input variables
- `terraform/outputs.tf`: Outputs (including Ansible inventory)
- `terraform/defaults.tfvars`: Default variable values

## Ansible Files
- `ansible/host_hardening.yml`: Harden host (compatible with Amazon Linux, Red Hat, Debian, Ubuntu)
- `ansible/hosts`: Inventory file (generated from Terraform output)
- `ansible/secrets.yaml`: Passwords/variables for playbook
- `ansible/sysconf_backup.sh`: Bash script for config backup

## Highlights
This working lab is to shape a better security posture with AWS deployments. Some detailed description of every steps in terraform and ansible blueprints.

### SSH hardening

The playbook ensure that PermitRootLogin is set to no, direct SSH access as root is blocked. This prevents attackers from targeting the root account, which is a common brute-force target. Moreover by setting PasswordAuthentication no, only key-based authentication is allowed. This eliminates the risk of password-guessing attacks over SSH, as attackers cannot use passwords to log in.

Benefits are:
- Only users with valid SSH keys can access the server.
- Even if an attacker knows a username or tries “root,” they cannot log in with a password.
- The attack surface for SSH is minimized, following best practices for secure remote access.


### Users configuration

The playbook sets passwords for ec2-user, root, and sysadmin using variables that are hashed with SHA-512. This ensures passwords are not stored in plain text and are strong. The Least Privilege Principle (LPP) is a foundational security concept stating that users, processes, and systems should be granted only the minimum privileges necessary to perform their intended function, and nothing more. In this configuration, least privilege is enforced by ensuring that only the dedicated sysadmin user is added to the administrative group (wheel or sudo), limiting privileged access to a clearly defined account. The Ansible playbook uses append: yes when managing group membership to avoid unintentionally escalating privileges by overwriting existing assignments. Additionally, requiring a password for sudo operations by ec2-user on Amazon Linux prevents silent or automated privilege escalation, ensuring that administrative actions are deliberate, authenticated, and auditable. Together, these measures reduce the attack surface and limit the potential impact of compromised accounts.

Benefits are:
- Controlled Privilege Escalation: Requiring a password for sudo by ec2-user ensures that administrative actions are always authenticated and auditable, preventing silent or automated privilege escalation.
- Reduced Attack Surface: Limiting admin access and enforcing authentication for privileged actions makes it harder for attackers to gain full control.

### Firewall configuration

For RedHat/Amazon Linux: firewalld is installed, enabled, and started. Only the SSH service is explicitly enabled, ensuring only SSH traffic is allowed by default.
For Debian/Ubuntu: UFW is installed, reset to defaults, incoming traffic is denied, outgoing is allowed, and only SSH (OpenSSH) is permitted. UFW is then enabled and restarted.

Benefits are:
- Only necessary ports (SSH) are open, minimizing the attack surface.
- Default-deny policy for incoming traffic blocks unwanted connections.
- Ensures a firewall is always active and persistent across reboots.

### Fail2ban configuration

Fail2Ban is implemented as a compensating security control to reduce the risk of brute-force attacks against SSH when stronger preventive measures (such as IP allowlisting, VPN-only access, or MFA) cannot be fully enforced. The configuration applies a findtime of 600 seconds, during which a maximum of 5 failed authentication attempts (maxretry) are allowed before the source IP is temporarily banned for 3600 seconds (bantime). This approach does not eliminate the underlying exposure of the SSH service but significantly increases the cost and difficulty of automated attacks, providing effective risk mitigation across both Debian- and RedHat-based systems.

Benefits are:
- Automated response: It reacts in real time to suspicious activity
- Reduces attack surface: By blocking malicious IPs, it limits the window of opportunity for attackers.

### SecurityGroup configuration

For best security, the security group should restrict SSH (port 22) access to only trusted IP addresses (e.g., your home or office IP), not 0.0.0.0/0. 
You can specify a single IP (e.g., 10.20.30.5/32) or a list of trusted IPs in the Terraform configuration using the `cidr_blocks` attribute.
Avoid exposing SSH to the entire internet unless absolutely necessary. Regularly review and update allowed IPs as  access requirements change over time. 

Benefits are:
- greatly reduces the risk of unauthorized access attempts from the internet.
- defense in depth for remote access.

## Lessons Learned

- limit admin access to only those who need it and enforce strong authentication for privileged actions.
- Defense in depth by combining security groups, host firewalls, SSH hardening, and intrusion prevention (Fail2Ban) for layered protection.
- Relevant /etc configurations backup to improve resilience of setup
