# Ansible Playbook for VPS Initial Configuration

This repository contains an Ansible playbook to configure and secure a VPS for initial deployment. It includes setting up an Ansible user, configuring SSH access, enabling a firewall, installing Docker, and connecting to Tailscale for secure networking.

---

## Requirements

### System Requirements
- Target VPS running a supported Linux distribution (Debian, Ubuntu, AlmaLinux, CentOS, or RedHat).
- Ansible installed on your local machine (control node).

### Dependencies
- `ansible-core` version 2.9 or higher.
- Python on the control node and target VPS.

---

## Setup Instructions

### 1. Install Ansible

Follow the [Ansible installation guide](https://docs.ansible.com/ansible/latest/installation_guide/index.html) to install Ansible on your local machine.

### 2. Clone the Repository

```bash
git clone https://github.com/jdam11/vps_initial_playbook.git
cd vps_initial_playbook
```

### 3. Update Inventory File

Create an inventory file (e.g., `inventory.ini`) and define your target VPS:

```ini
[vps_init]
<your_vps_hostname> ansible_host=<your_vps_ip> ansible_user=<initial_user> ansible_password=<password> ansible_port=<ssh port>
```
For the initial configuration, use the IP address, username, password, and SSH port provided by your VPS. Once you've created your own user and configured Tailscale (if enabled), the inventory file will automatically update with the new information to ensure continued access.

### 4. Customize Variables

Edit the `vars` section of the playbook or pass them as extra variables. Here are the required variables:

- `tailscale_auth_key`: Tailscale auth key (replace `<your_tailscale_auth_key>` in the playbook).
- `tailscale_subnets`: Subnets to advertise (e.g., `['0.0.0.0/0', '::/0']` or leave empty).
- `configure_tailscale`: Set to `true` to enable Tailscale setup (default).
- `expose_ssh_port`: Set to `true` to expose SSH to the public internet (default: `false`).
- `ansible_user_name`: Name of the Ansible user to create.
- `ansible_user_password`: Password for the Ansible user (hashed automatically).
- `ansible_user_ssh_key`: Public SSH key for the Ansible user.

Example to override variables:

```yaml
vars:
  tailscale_auth_key: "tskey-xxxxxxxx"
  configure_tailscale: true
  expose_ssh_port: true
  ansible_user_name: "ansible_user"
  ansible_user_password: "StrongPassword123"
  ansible_user_ssh_key: |
    ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAw3...
```

### 5. Run the Playbook

Run the playbook with the following command:

Note: If you plan to use Tailscale exclusively for SSH access, keep `expose_ssh_port` variable set to false.

```bash
ansible-playbook -i inventory.ini playbook.yml --ask-become-pass
```

Use the `--extra-vars` flag to override variables dynamically:

```bash
ansible-playbook -i inventory.ini playbook.yml --ask-become-pass \
  --extra-vars "tailscale_auth_key=tskey-xxxxxxxx"
```

---

## Features

### 1. User Management
- Creates an Ansible user with sudo privileges.
- Configures SSH access with a public key.
- Enforces SSH security by disabling password authentication and root login.

### 2. Firewall Setup
- Installs and configures UFW (Uncomplicated Firewall).
- Allows specific ports (e.g., SSH, HTTP, HTTPS) as defined in `tcp_ports` and `udp_ports` variables.

### 3. Tailscale VPN
- Installs and configures Tailscale for secure networking.
- Tailscale eliminates the need for public-facing SSH, adding another layer of security by providing secure remote access over a private network.
- Advertises custom subnets if provided.

### 4. Docker Installation
- Installs Docker CE.
- Configures Docker in rootless mode for security.
- Prepares a `/docker` directory for Docker usage.

---

## Customization

- **Update Variables**: Modify variables in the `vars` section of the playbook to suit your setup.
- **Additional Ports**: Add more ports to `tcp_ports` and `udp_ports` as needed.
- **Tailscale Configuration**: Customize the Tailscale setup for specific networking needs.

---

## Validation

After running the playbook, verify the configuration:

1. **SSH Access**:
   - Test login with the new Ansible user and SSH key.
   - This step may be skipped if Tailscale is used exclusively for remote access.
   - Confirm the custom SSH port is active if exposed.

2. **Firewall**:
   - Check UFW status: `sudo ufw status`.

3. **Tailscale**:
   - Verify connection with: `tailscale status`.

4. **Docker**:
   - Test Docker functionality:

```bash
sudo -u docker docker run hello-world
```

---

## Troubleshooting

- Use the `-vvv` flag with `ansible-playbook` for detailed output during execution. For example, you can use `ansible-playbook -i inventory.ini playbook.yml -vvv` to debug issues such as verifying SSH connectivity or checking if the Tailscale setup is working correctly.
- Ensure the initial SSH user has passwordless sudo access.
- Verify Tailscale auth key validity if connection fails.

---

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.

---

## Contributions

Contributions, issues, and feature requests are welcome! Feel free to open a PR or issue.

---

## Acknowledgments

Thanks to the open-source community for providing the tools and resources that make projects like this possible.

