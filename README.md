# EC2 + SSH from macOS (Step-by-step)

This guide walks you through:
1) creating an EC2 instance in AWS  
2) setting up the key pair + security group to allow SSH  
3) logging in via SSH from macOS using the downloaded `.pem` key  

---

## 0) Prerequisites (AWS + macOS)

### On AWS
- You have an AWS account
- You can create/select an EC2 instance
- You have permission to manage EC2 key pairs and security groups

### On macOS
- You have an internet connection
- You know your desired AWS region (example: `us-east-1`)

---

## 1) Install the necessary command-line tools (macOS)

You’ll mainly need:
- **AWS CLI** (optional but helpful for verification/fetching info)
- **OpenSSH client** (usually already installed)
- **ssh-keygen** (usually already installed)

### Check OpenSSH + ssh-keygen
Run:
```bash
ssh -V
ssh-keygen -V
```

If those commands exist, you’re good.

### Install AWS CLI (recommended)

#### Option A (Homebrew)
```bash
brew install awscli
```

#### Option B (manual install)
- Download and install **AWS CLI v2** from: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

Verify:
```bash
aws --version
```

---

## 2) Configure AWS CLI (so `aws ...` commands work)

If you plan to run AWS CLI commands (like `aws sts get-caller-identity`), configure credentials + region:

1) Run:
```bash
aws configure
```

2) Enter:
- `AWS Access Key ID`
- `AWS Secret Access Key`
- `Default region name` (example: `us-east-1`)
- `Default output format` (example: `json`)

3) Verify it works:
```bash
aws sts get-caller-identity
```

If it returns your AWS identity (account/ARN/user), setup is working.

> Tip: Don’t upload or commit your credentials. Prefer IAM users/roles configured securely for your environment.

---

## 3) Create an EC2 instance (AWS Console)

### Step 3.1 — Launch an instance
1. Open the AWS Console
2. Go to **EC2** → **Instances**
3. Click **Launch instances**
4. Choose an **AMI** (example choices):
   - **Amazon Linux**
   - **Ubuntu Server**

### Step 3.2 — Choose instance type
- For learning/testing: `t2.micro` or `t3.micro` (often free tier)

### Step 3.3 — Key pair (critical for SSH)
1. Click **Create new key pair**
2. Name it (example: `my-ec2-key`)
3. Select key format: **.pem**
4. Click **Create key pair**
5. Download the `.pem` file and **keep it safe**

> If you lose the `.pem`, you can’t use it again. You’ll need a new key pair.

### Step 3.4 — Network settings (must allow SSH)
In **Security group**, ensure an inbound rule includes:
- **Type:** SSH
- **Port:** 22
- **Source:** your IP (recommended)  
  or `0.0.0.0/0` (less secure)

### Step 3.5 — Launch
Click **Launch instance** and wait until the instance is **Running**.

---

## 4) Identify the SSH username (depends on AMI)

Common defaults:
- **Amazon Linux / Amazon Linux 2:** `ec2-user`
- **Ubuntu:** `ubuntu`
- **Debian:** often `admin` or `debian` (varies)

If unsure, open AWS Console and use the **Connect** instructions for your instance.

---

## 5) Move your `.pem` key to the right place on macOS

Assume your downloaded key is:
`~/Downloads/my-ec2-key.pem`

Move it into `~/.ssh/`:
```bash
mkdir -p ~/.ssh
mv ~/Downloads/my-ec2-key.pem ~/.ssh/
```

Set correct permissions for SSH:
```bash
chmod 400 ~/.ssh/my-ec2-key.pem
```

### Why `chmod 400` matters
SSH refuses to use private keys that are “too open” (readable by group/others).  
- `400` means: **read-only for you (the owner)** and **no permissions** for group/others.
- If permissions are wrong, you may get errors like **“UNPROTECTED PRIVATE KEY FILE”** or **“Permission denied (publickey)”**.

Check permissions:
```bash
ls -l ~/.ssh/my-ec2-key.pem
```

---

## 6) Get the EC2 public IP address

In AWS Console:
- EC2 → Instances → select your instance
- Copy **Public IPv4 address**

---

## 7) SSH into the instance from macOS

### SSH command template
```bash
ssh -i ~/.ssh/my-ec2-key.pem <username>@<public-ip>
```

Examples:

**Amazon Linux:**
```bash
ssh -i ~/.ssh/my-ec2-key.pem ec2-user@YOUR_PUBLIC_IP
```

**Ubuntu:**
```bash
ssh -i ~/.ssh/my-ec2-key.pem ubuntu@YOUR_PUBLIC_IP
```

### First-time connection prompt
If you see: “Are you sure you want to continue connecting?”
- Type `yes`

---

## 8) Common SSH problems (and fixes)

### Problem A — “Permission denied (publickey)”
Check in order:
- You used the right `.pem` file for that instance/key pair
- You used the correct username (`ec2-user` vs `ubuntu`)
- Key permissions:
  ```bash
  chmod 400 ~/.ssh/my-ec2-key.pem
  ```
- Confirm you moved/renamed the right file:
  ```bash
  ls -l ~/.ssh/my-ec2-key.pem
  ```

### Problem B — “Connection timed out”
Usually security group / networking:
- In EC2 **Security Group** inbound rules: confirm SSH **port 22**
- Confirm **Source** is your current public IP (or a narrower range)

### Problem C — “Host key verification failed”
If the instance was replaced and the host key changed:
```bash
ssh-keygen -R YOUR_PUBLIC_IP
```
Then try SSH again.

---

## 9) Useful Linux command-line tools (inside the EC2 instance)

Commands vary by distro:

### Amazon Linux (yum)
```bash
sudo yum update -y
sudo yum install -y curl wget vim htop
```

### Ubuntu/Debian (apt)
```bash
sudo apt update -y
sudo apt install -y curl wget vim htop
```

Useful basics:
- `whoami`
- `uname -a`
- `ip a`
- `df -h`
- `free -h`
- `systemctl status <service>` (if systemd-based)

---

## 10) Optional: Use AWS CLI to fetch the public IP

If you know your `INSTANCE_ID`:
```bash
aws ec2 describe-instances \
  --instance-ids INSTANCE_ID \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text
```

---

## 11) Quick reference (copy/paste)

1) Key permissions:
```bash
chmod 400 ~/.ssh/my-ec2-key.pem
```

2) SSH (Amazon Linux):
```bash
ssh -i ~/.ssh/my-ec2-key.pem ec2-user@YOUR_PUBLIC_IP
```

3) SSH (Ubuntu):
```bash
ssh -i ~/.ssh/my-ec2-key.pem ubuntu@YOUR_PUBLIC_IP
```

---

## Notes
- Keep the `.pem` file private. Do **not** commit it to Git.
- If you stop/recreate the instance, the public IP and/or host key may change; you may need to update `known_hosts`.

---

## 12) Preview / “test” this README locally (macOS)

A README isn’t executable code, so “testing” means verifying formatting and that commands are usable.

### Option A — Preview in VS Code (recommended)
1. Open `README.md` in VS Code
2. Use **Markdown Preview**
   - Press **Shift + Command + P**
   - Type: `Markdown: Open Preview`
   - Press Enter

### Option B — Convert to HTML and open in a browser
1. Install Pandoc:
   ```bash
   brew install pandoc
   ```
2. Convert:
   ```bash
   pandoc README.md -o README.html
   ```
3. Open:
   ```bash
   open README.html
   ```

### Option C — Validate shell commands (manual)
- Copy/paste commands one at a time and run them.
- Skip anything that uses AWS credentials until your AWS CLI is configured correctly.
