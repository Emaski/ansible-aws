# EC2 Instance Provisioning with Ansible

An Ansible playbook that creates an AWS EC2 key pair, saves the private key locally, and launches a single `t2.micro` instance in `us-west-2`.

## Overview

The playbook runs against `localhost`, so AWS API calls are made from your machine using your configured AWS credentials. It performs three tasks:

1. **Create a key pair** named `sample` in `us-west-2` using the `ec2_key` module.
2. **Save the private key** to `./sample.pem` (only when a new key is created).
3. **Launch an EC2 instance** named `public-compute-instance` using the `sample` key pair, the `default` security group, and the AMI `ami-02d8bad0a1da4b6fd`.

## Prerequisites

- Python 3 and [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- AWS credentials configured locally (via `aws configure`, environment variables, or an AWS profile)
- The following Ansible collections:

```bash
ansible-galaxy collection install amazon.aws community.aws
```

- Python `boto3` and `botocore`:

```bash
pip install boto3 botocore
```

## Usage

Run the playbook:

```bash
ansible-playbook provision.yml
```

Once the instance is running, connect with the saved key:

```bash
chmod 400 sample.pem
ssh -i sample.pem ec2-user@<public-ip>
```

Replace `ec2-user` with the default username for your AMI (for example, `ubuntu` for Ubuntu images).

## Configuration

Values you may want to change in `provision.yml`:

| Setting | Current value | Description |
|---|---|---|
| `region` | `us-west-2` | AWS region for the key pair and instance |
| `name` (key) | `sample` | Name of the EC2 key pair |
| `name` (instance) | `public-compute-instance` | Instance name |
| `instance_type` | `t2.micro` | EC2 instance size |
| `image_id` | `ami-02d8bad0a1da4b6fd` | AMI ID (region-specific) |
| `security_group` | `default` | Security group to attach |
| `exact_count` | `1` | Number of matching instances to maintain |

## Notes and Caveats

- **Private key security:** `sample.pem` contains a sensitive credential. Do not commit it to version control. Add it to `.gitignore`:


- **Security group:** The `default` security group usually does not allow inbound SSH from the internet. Add an inbound rule for port 22 from your IP if you need to connect.
- **AMI IDs are region-specific** and change over time. Verify the AMI exists in your region before running.
- **Idempotency:** `exact_count: 1` prevents duplicate instances, but the key pair task only returns a private key the first time it is created. If the key already exists in AWS, you need the original `sample.pem` file.
- **Parameter naming:** Newer versions of the `amazon.aws` collection prefer `security_groups` (a list) over `security_group`. Check your collection version if you see deprecation warnings.
- **Cost:** Running EC2 instances incurs charges. Terminate the instance when you're finished testing.

## Cleanup

To terminate the instance, delete it from the AWS console or run a playbook with `state: absent`. To delete the key pair:

```bash
aws ec2 delete-key-pair --key-name sample --region us-west-2
```

