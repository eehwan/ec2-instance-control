# ec2ctl: Effortless EC2 Instance Control

![Python Version](https://img.shields.io/badge/python-3.7+-blue.svg) ![PyPI Version](https://img.shields.io/pypi/v/ec2ctl.svg) ![License](https://img.shields.io/badge/license-MIT-green.svg)

Read this in other languages: [한국어](https://github.com/eehwan/ec2-control/blob/main/README_ko.md) | [English](https://github.com/eehwan/ec2-control/blob/main/README.md)

---

**Tired of logging into AWS Console just to start your EC2 instance and copy the IP again?**

**`ec2ctl` is a lightweight CLI tool built for developers who use EC2 as a dev environment — turning it on/off frequently to save costs — and want a faster, simpler way to manage it.**  
No more digging through the console or copying dynamic IPs manually.

## Table of Contents

- [Purpose](#purpose)
- [Features](#features)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
  - [`ec2ctl init`](#ec2ctl-init)
  - [`ec2ctl list`](#ec2ctl-list)
  - [`ec2ctl start`](#ec2ctl-start)
  - [`ec2ctl stop`](#ec2ctl-stop)
  - [`ec2ctl status`](#ec2ctl-status)
  - [`ec2ctl connect`](#ec2ctl-connect)
- [Error Handling & Troubleshooting](#error-handling--troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Purpose

Many developers use EC2 as a lightweight development server — spinning it up only when needed to reduce AWS bills.  
But starting/stopping instances manually via the AWS Console can be a hassle, and changing public IPs makes SSH access even more tedious.

`ec2ctl` eliminates that friction. It lets you:

- Start/stop EC2 instances with a single, memorable command
- Connect via SSH without worrying about changing IPs
- Group and manage instances in a config file for fast lookup
- Skip the AWS Console entirely for routine tasks

All with a simple YAML config and clean CLI UX.

## Features

- **Intuitive Commands:** `ec2ctl start dev-server`, `ec2ctl stop all`, `ec2ctl status backend-group`.
- **Flexible Configuration:** Manage instances by name or group using a simple `config.yaml` file.
- **Enhanced User Experience:** Supports `--dry-run`, `--verbose`, and `--yes` options.
- **Robust Error Handling:** Provides clear messages for AWS authentication, instance state, and configuration issues.
- **SSH Connection:** Connect directly to instances, with optional automatic stopping on disconnect.

## Installation

### Prerequisites

- Python 3.7+
- `pip` (Python package installer)
- AWS CLI configured with your credentials (`aws configure`)

### Install `ec2ctl`

You can install `ec2ctl` directly from PyPI using pip:

```bash
pip install ec2ctl
```

#### For Development

If you plan to contribute or modify the source code, you can install it in editable mode:

```bash
git clone https://github.com/eehwan/ec2-control.git
cd ec2-control
pip install -e .
```
This allows changes to the source code to be immediately reflected without reinstallation.

## Configuration

`ec2ctl` uses a YAML configuration file located at `~/.ec2ctl/config.yaml`.
The `ec2ctl init` command now interactively guides you through selecting an AWS profile and region, then automatically discovers your EC2 instances and generates a `config.yaml` based on them.

```bash
ec2ctl init
```

### Generated `config.yaml` Structure

The `ec2ctl init` command discovers your EC2 instances and generates a `config.yaml` based on them. It will look similar to this:

```yaml
default_profile: your-selected-profile
default_region: your-selected-region

instances:
  web-1:
    id: i-0123456789abcdef0
    ssh_user: ec2-user # Auto-guessed, verify and change if needed
    ssh_key_path: ~/.ssh/web-1-key.pem # Auto-guessed, verify and change if needed
  web-2:
    id: i-fedcba9876543210
    ssh_user: ubuntu # Auto-guessed, verify and change if needed
    ssh_key_path: ~/.ssh/web-2-key.pem # Auto-guessed, verify and change if needed
  # ... other discovered instances
```

-   `default_profile`: Your default AWS profile name, selected during `init`.
-   `default_region`: Your default AWS region, selected during `init`.
-   `instances`: A map of discovered EC2 instance names (or IDs if no Name tag) to their details.
    -   `id`: The EC2 instance ID.
    -   `ssh_user`: An auto-guessed SSH user (defaults to `ec2-user`). **You must verify and change this based on your instance's AMI (e.g., `ubuntu` for Ubuntu AMIs).**
    -   `ssh_key_path`: An auto-guessed path to your SSH private key, based on the key pair name linked to the instance. **You must verify this path and ensure it points to your actual private key file.** If no key name was found, a placeholder will be used.

**Manually Adding Instances or Groups:**

After `ec2ctl init` generates the initial config, you can manually edit the `~/.ec2ctl/config.yaml` file to add more instances or define groups. Refer to `config.example.yaml` for examples of how to define instance groups or simple ID entries.

```yaml
# Example entries from config.example.yaml that you can add manually:
instances:
  # ... (existing discovered instances)

  dev-server:
    id: i-0abc1234567890
    ssh_user: ec2-user
    ssh_key_path: ~/.ssh/my-key.pem
    ssh_port: 2222
  backend-api:
    - id: i-01aaa111aaa
      ssh_user: ubuntu
      ssh_key_path: ~/.ssh/backend_key.pem
    - id: i-01bbb222bbb
      ssh_user: ubuntu
      ssh_key_path: ~/.ssh/backend_key.pem
  staging: i-0123staging456
```

## Usage

All commands support `--profile`, `--region`, `--dry-run`, and `--verbose` options. Commands that modify state also support `--yes` (`-y`).

### `ec2ctl init [--yes]`

Initializes the config file by discovering EC2 instances from your AWS account. It will prompt you to select an AWS profile and region.

```bash
ec2ctl init
# Overwrite without confirmation
ec2ctl init --yes
```

### `ec2ctl list`

Lists all EC2 instances and groups configured in `~/.ec2ctl/config.yaml`.

```bash
ec2ctl list
```

### `ec2ctl start [name|group]`

Starts the specified EC2 instance(s).

```bash
ec2ctl start dev-server
ec2ctl start backend-api
```

### `ec2ctl stop [name|group]`

Stops the specified EC2 instance(s).

```bash
ec2ctl stop dev-server
ec2ctl stop backend-api
```

### `ec2ctl status [name|group|all]`

Gets the current status of the specified EC2 instance(s).

```bash
ec2ctl status dev-server
ec2ctl status all
```

### `ec2ctl connect [name] [--user USER] [--key KEY_PATH] [--port PORT] [--keep-running]`

Connects to an EC2 instance via SSH, starting it if necessary. By default, the instance will be stopped when the SSH session disconnects.

-   `name`: The name of the instance or group as defined in `config.yaml`.
-   `--user USER`: Override the SSH user defined in config.
-   `--key KEY_PATH`: Override the path to the SSH private key file defined in config.
-   `--port PORT`: Override the SSH port defined in config.
-   `--keep-running`: Keep the instance running after the SSH session disconnects.

```bash
# Connect to dev-server, stop on disconnect (default)
ec2ctl connect dev-server

# Connect to dev-server with a specific port
ec2ctl connect dev-server --port 2222

# Connect to dev-server, keep running on disconnect
ec2ctl connect dev-server --keep-running

# Connect with overridden user and key
ec2ctl connect dev-server --user admin --key ~/.ssh/my_custom_key.pem
```

## Error Handling & Troubleshooting

`ec2ctl` provides informative error messages for common issues:

-   **Config file not found:** Run `ec2ctl init` to create the default configuration.
-   **Instance/Group not found:** Ensure the name is correctly spelled and defined in `config.yaml`.
-   **AWS Authentication/Authorization issues:** Check your AWS CLI configuration (`aws configure`) and IAM policies (e.g., `ec2:StartInstances`, `ec2:StopInstances`, `ec2:DescribeInstances`).
-   **Incorrect Instance State:** Attempting to start an already running instance, or stop an already stopped instance.
-   **SSH Connection Issues:** Ensure the instance has a public IP, security groups allow SSH (port 22), and the SSH key path/permissions are correct.

## Contributing

Contributions are welcome! Please feel free to open issues or submit pull requests.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.