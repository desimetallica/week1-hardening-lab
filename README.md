# week1-hardening-lab

## Objective

- Harden Linux (SSH, users, firewall, permissions)
- Simple bash script for config backup
- Deploy EC2 on AWS (free tier)
- Document objectives, environment, steps, config, and lessons learned
- Folders: notes, config, screenshots, terraform, ansible

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

## Notes
- Use a free tier-eligible AMI (e.g., Amazon Linux 2/2023, Ubuntu 22.04, etc.).
- Default SSH user: `ec2-user` (Amazon Linux), `ubuntu` (Ubuntu), `admin`/`root` (others).
- Security group allows SSH (port 22) from your IP.
- The playbook configures firewall, disables root SSH login, and sets up Fail2Ban.
- For Red Hat/Amazon Linux, firewalld is used; for Debian/Ubuntu, UFW is used.

## Lessons Learned
- Automating infrastructure and hardening increases security and repeatability.
- Always check AMI and instance type compatibility for your AWS region.
- Use variables and tfvars files to simplify deployments and avoid exposing secrets.
- Configured a new AWS free tier account
- Created an admin user with Key
- Installed AWS cli and tested it
- Configured a simple AWS EC2 Instance IaC template
- Configured a simple AWS EC2 Key IaC template
