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
git clone https://github.com/<your-username>/<your-repo-name>.git
cd <your-repo-name>
```

### 3. Update Inventory File

Create an inventory file (e.g., `inventory.yml`) and define your target VPS:

```yaml
all:
  hosts:
    vps_init:
      ansible_host: <your_vps_ip>
      ansible_user: <initial_user>
      ansible_ssh_private_key_file: <path_to_your_ssh_private_key>
```

### 4. Customize Variables

Edit the `vars` section of the playbook or pass them as extra variables. Note: The `ssh_port` variable is only necessary if you want to expose SSH via the public IP. With Tailscale installed, this might not be required for all use cases. Here are the required variables:

- `tailscale_auth_key`: Tailscale auth key (replace `<your_tailscale_auth_key>` in the playbook).
- `tailscale_subnets`: Subnets to advertise (e.g., `['0.0.0.0/0', '::/0']` or leave empty).
- `ssh_port`: Custom SSH port. This is primarily for cases where SSH needs to be accessible via the public IP. It is optional when using Tailscale.
- `ansible_user_name`: Name of the Ansible user to create.
- `ansible_user_password`: Password for the Ansible user (hashed automatically).
- `ansible_user_ssh_key`: Public SSH key for the Ansible user.

Example to override variables:

```yaml
vars:
  tailscale_auth_key: "tskey-xxxxxxxx"
  ssh_port: 2222
  ansible_user_name: "ansible_user"
  ansible_user_password: "StrongPassword123"
  ansible_user_ssh_key: |
    ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAw3...
```

### 5. Run the Playbook

Run the playbook with the following command:

Note: If you plan to use Tailscale exclusively for SSH access, you can skip setting the `ssh_port` variable.

```bash
ansible-playbook -i inventory.yml playbook.yml --ask-become-pass
```

Use the `--extra-vars` flag to override variables dynamically:

```bash
ansible-playbook -i inventory.yml playbook.yml --ask-become-pass \
  --extra-vars "tailscale_auth_key=tskey-xxxxxxxx ssh_port=2222"
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

Note: If you plan to access the server exclusively through Tailscale, you can skip the `ssh_port` validation step.

1. **SSH Access**:
   - Test login with the new Ansible user and SSH key.
   - This step may be skipped if Tailscale is used exclusively for remote access.
   - Confirm the custom SSH port is active.

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

- Use the `-vvv` flag with `ansible-playbook` for detailed output during execution. For example, you can use `ansible-playbook -i inventory.yml playbook.yml -vvv` to debug issues such as verifying SSH connectivity or checking if the Tailscale setup is working correctly.
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

