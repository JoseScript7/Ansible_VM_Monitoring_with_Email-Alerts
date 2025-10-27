# Ansible_VM_Health_Monitoring

A simple Ansible playbook designed to monitor various aspects of your virtual machines' health, including system uptime, memory usage, disk usage, active processes, network activity, and the number of current logged-in users. This playbook is useful for administrators or students who want to get a quick overview of multiple VMs' performance and health.

## Features

This playbook performs several health checks on your virtual machines:

- **System Uptime**: Displays how long the system has been running since the last reboot.
- **CPU Cores**: Reports the number of CPU cores available on the system.
- **Memory Usage**: Shows the total, used, and available memory in a human-readable format.
- **Disk Space Usage**: Displays the total, used, and remaining disk space for the root filesystem (/).
- **Active Processes**: Counts and reports the number of running processes.
- **Current Logged-in Users**: Displays the number of users currently logged into the system.
- **Network Activity**: Displays the number of active network connections.
- **Email Reports**: Automatically sends consolidated health reports to your email.

## Requirements

This playbook should work on most Linux-based systems and assumes that the following are available:

**Control Machine (where you run Ansible):**
- Ansible 2.9 or higher
- Python 3.6 or higher

**Target VMs:**
- uptime
- nproc
- free
- df
- ps
- who
- ss (or netstat)

If you are using a minimal Linux distribution, make sure these commands are installed.

## Usage

**Clone the repository:**
```bash
git clone https://github.com/JoseScript7/Ansible_VM_Monitoring_with_Email-Alerts.git
```

**Navigate to the project directory:**
```bash
cd Ansible_VM_Monitoring_with_Email-Alerts
```

**Configure your VMs:**

Edit `inventory.ini` and add your VM details:
```bash
nano inventory.ini
```

**Setup SSH access:**
```bash
ssh-copy-id root@192.168.1.10
ssh-copy-id root@192.168.1.11
```

**Configure email settings:**

Edit `health_check_email.yml` and update your email credentials (lines 51-53)

**Test connection:**
```bash
ansible all -i inventory.ini -m ping
```

**Run the playbook:**
```bash
ansible-playbook -i inventory.ini health_check_email.yml
```

## Ansible Playbook Written:



## Output



## How the Playbook Works

The playbook consists of two main plays that work together to monitor VMs and send email reports.

### Play 1: Check Health of All VMs

| Ansible Component | Purpose | Details |
|-------------------|---------|---------|
| `hosts: all` | Target selection | Runs on all VMs listed in inventory.ini |
| `become: true` | Privilege escalation | Execute commands with root/sudo access |
| `tasks:` | Task list | Defines the jobs to execute |

**Task Breakdown:**

| Task Name | Module Used | What It Does | Output |
|-----------|-------------|--------------|--------|
| Run health check script | `shell` | Executes bash script on each VM to collect health metrics | Stores output in `vm_health` variable |
| Save individual VM reports | `copy` | Creates text file with VM health data on your local machine | Files: `/tmp/health_vm1.txt`, `/tmp/health_vm2.txt`, etc. |

**Key Ansible Concepts:**

| Concept | Code Example | Explanation |
|---------|--------------|-------------|
| **Register** | `register: vm_health` | Saves task output to reuse in later tasks |
| **Variable Access** | `{{ vm_health.stdout }}` | Gets the text output from the health check |
| **Inventory Variable** | `{{ inventory_hostname }}` | Gets VM name (vm1, vm2, vm3...) |
| **Delegate** | `delegate_to: localhost` | Runs this task on your computer instead of the VM |

---

### Play 2: Email All Reports

| Ansible Component | Purpose | Details |
|-------------------|---------|---------|
| `hosts: localhost` | Target selection | Runs only on your local machine |
| `gather_facts: yes` | System information | Collects date/time for report timestamp |
| `vars:` | Variables | Stores email credentials and recipient |

**Email Configuration:**

| Variable | Purpose | Example |
|----------|---------|---------|
| `gmail_user` | Sender email address | `"your.email@gmail.com"` |
| `gmail_app_password` | Gmail app password (16 digits) | `"abcd efgh ijkl mnop"` |
| `send_to` | Recipient email address | `"recipient@example.com"` |

**Task Breakdown:**

| Task # | Task Name | Module | What It Does | Where It Runs |
|--------|-----------|--------|--------------|---------------|
| 1 | Create combined report | `shell` | Merges all VM reports into `/tmp/final_report.txt` | Your computer |
| 2 | Send email with report | `shell` (Python) | Sends email via Gmail SMTP server | Your computer |
| 3 | Show email status | `debug` | Displays success/error message in terminal | Your computer |
| 4 | Show report location | `debug` | Shows where report is saved locally | Your computer |

**Python Email Process:**

| Step | Action | Technical Details |
|------|--------|-------------------|
| 1 | Read report file | Opens `/tmp/final_report.txt` |
| 2 | Configure email | Sets sender, recipient, subject |
| 3 | Create message | Formats email with report content |
| 4 | Connect to SMTP | Connects to `smtp.gmail.com:587` |
| 5 | Authenticate | Logs in with Gmail credentials |
| 6 | Send email | Transmits email message |
| 7 | Disconnect | Closes SMTP connection |

---

### Ansible Modules Used

| Module | Purpose | Used In |
|--------|---------|---------|
| `shell` | Execute shell commands/scripts | Health check script, Combine reports, Send email |
| `copy` | Create or copy files | Save VM reports locally |
| `debug` | Print messages to terminal | Show status messages |

**Special Ansible Features:**

| Feature | Syntax | Purpose |
|---------|--------|---------|
| Register | `register: variable_name` | Save task output for later use |
| Variable interpolation | `{{ variable_name }}` | Insert variable values |
| Delegation | `delegate_to: localhost` | Run task on different machine |
| Jinja2 templating | `{{ vm_health.stdout }}` | Access nested data |

---

## Playbook Execution Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                    START: ansible-playbook                       │
│                 health_check_email.yml -i inventory.ini          │
└────────────────────────────────┬────────────────────────────────┘
                                 │
                                 ▼
                    ┌────────────────────────┐
                    │  Read inventory.ini    │
                    │  VMs: vm1, vm2...vm10  │
                    └────────────┬───────────┘
                                 │
                                 ▼
                    ┌────────────────────────┐
                    │ Setup SSH Connection   │
                    │ Using SSH Keys         │
                    │ (ssh-keygen generated) │
                    └────────────┬───────────┘
                                 │
                    ┌────────────┴────────────┐
                    │                         │
        ┌───────────▼────────┐    ┌──────────▼──────────┐
        │   PLAY 1           │    │                     │
        │ Check All VMs      │    │  Parallel Execution │
        │                    │    │  All VMs at once    │
        └───────────┬────────┘    └─────────────────────┘
                    │
        ┌───────────┴───────────────────────────┐
        │                                       │
        ▼                                       ▼
┌───────────────┐                      ┌────────────────┐
│   Task 1      │                      │   Task 1       │
│   VM1         │                      │   VM2          │
│ Run Health    │  ...                 │ Run Health     │  ...
│ Check Script  │                      │ Check Script   │
└───────┬───────┘                      └────────┬───────┘
        │                                       │
        │  Output: System metrics               │
        │  register: vm_health                  │
        │                                       │
        ▼                                       ▼
┌───────────────┐                      ┌────────────────┐
│   Task 2      │                      │   Task 2       │
│Save Report    │                      │Save Report     │
│/tmp/health_   │                      │/tmp/health_    │
│vm1.txt        │                      │vm2.txt         │
└───────┬───────┘                      └────────┬───────┘
        │                                       │
        └───────────┬───────────────────────────┘
                    │
                    │ delegate_to: localhost
                    │ (Saves on YOUR computer)
                    │
                    ▼
        ┌───────────────────────┐
        │  All VM reports saved │
        │  on your local machine│
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │      PLAY 2           │
        │   Email Reports       │
        │   (localhost only)    │
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │   Task 3              │
        │ Combine all reports   │
        │ into final_report.txt │
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────────────────┐
        │          Task 4                   │
        │      Send Email via Python        │
        │  ┌─────────────────────────────┐  │
        │  │ 1. Read final_report.txt    │  │
        │  │ 2. Connect to Gmail SMTP    │  │
        │  │ 3. Login with credentials   │  │
        │  │ 4. Send email               │  │
        │  │ 5. Close connection         │  │
        │  └─────────────────────────────┘  │
        └───────────┬───────────────────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │   Task 5 & 6          │
        │ Show Success Message  │
        │ Display Report Path   │
        └───────────┬───────────┘
                    │
                    ▼
        ┌───────────────────────┐
        │    END: Success       │
        │ ✓ Email sent          │
        │ ✓ Reports saved       │
        └───────────────────────┘
```

---

## SSH Connection Setup

This playbook uses **SSH key-based authentication** (not ssh-copy-id).

**Setup Process:**

```bash
# 1. Generate SSH key pair on your computer
ssh-keygen -t rsa -b 4096

# 2. Copy public key to each VM manually
cat ~/.ssh/id_rsa.pub | ssh root@192.168.1.10 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# 3. Set correct permissions on VM
ssh root@192.168.1.10 "chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"

# 4. Test connection (should not ask for password)
ssh root@192.168.1.10

# 5. Configure inventory.ini to use the key
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

**How Ansible Uses SSH Keys:**

| Step | What Happens |
|------|--------------|
| 1. Ansible reads inventory.ini | Gets VM IPs and SSH key path |
| 2. For each VM | Establishes SSH connection using private key |
| 3. Authentication | VM verifies using public key in authorized_keys |
| 4. Connection established | Ansible can now execute commands |
| 5. Parallel execution | All VMs are accessed simultaneously |

## Contributions

Feel free to contribute to this project in the following ways:

- **Fork the repository**: Create a personal copy of the repository and make changes that you think could improve the playbook.
- **Report issues**: If you find bugs or problems with the playbook, please create an issue in the repository with a description of the problem and how to reproduce it.
- **Suggest features**: If you have ideas for new features or improvements to the playbook, feel free to open an issue or create a pull request.
- **Submit pull requests**: If you've fixed a bug or added a new feature, please submit a pull request.
