# EC2 + SSH from macOS (Step-by-step)

This guide walks you through:
1) creating an EC2 instance in AWS  
2) setting up the key pair + security group to allow SSH  
3) logging in via SSH from macOS using the `.pem` key

---

## 0) Prerequisites (AWS + macOS)

### On AWS
- You have an AWS account
- You can create/select an EC2 instance
- You have permission to manage EC2 key pairs and security groups

### On macOS
- You have an internet connection
- You have the AWS region you want to use (e.g., `us-east-1`)

---

## 1) Install the necessary command-line tools (macOS)

You’ll mainly need:
- **AWS CLI** (optional but helpful)
- **OpenSSH client** (usually already installed on macOS)
- **ssh-keygen** (usually already installed)

### Check OpenSSH + ssh-keygen
Run:
```bash
ssh -V
ssh-keygen -V
```

If those commands exist, you’re good.

### Install AWS CLI (recommended)
Option A (Homebrew):
```bash
brew install awscli
```

Option B (manual install)
- Download and install **AWS CLI v2** from: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

Verify:
```bash
aws --version
```

---

## 2) Create an EC2 instance (AWS Console)

### Step 2.1 — Launch an instance
1. Open the AWS Console
2. Go to **EC2** → **Instances**
3. Click **Launch instances**
4. Choose an **AMI** (example choices):
   - **Amazon Linux** (common default)
   - **Ubuntu Server** (also common)

### Step 2.2 — Choose instance type
- For learning/testing: `t2.micro` or `t3.micro` (free tier often available)

### Step 2.3 — Key pair (critical for SSH)
1. Click **Create new key pair**
2. Name it (example: `my-ec2-key`)
3. Select key format: **.pem**
4. Click **Create key pair**
5. Download the `.pem` file and **keep it safe**

> If you lose the `.pem`, you can’t use it again. You’ll need a new key pair.

### Step 2.4 — Network settings (must allow SSH)
1. In **Security group**, either:
   - create a new security group, or
   - edit an existing one
2. Ensure there is an inbound rule for:
   - **Type:** SSH
   - **Port:** 22
   - **Source:** your IP (recommended)  
     - or `0.0.0.0/0` (less secure)

### Step 2.5 — Launch
Click **Launch instance** and wait for the instance to reach **Running**.

---

## 3) Identify how to log in (EC2 default username)

The SSH “username” depends on the AMI:

- **Amazon Linux / Amazon Linux 2:** usually `ec2-user`
- **Ubuntu:** usually `ubuntu`
- **Debian:** usually `admin` or `debian` (varies)

If unsure, check the AMI documentation or the “Connect” instructions inside AWS Console for your instance.

---

## 4) Move your `.pem` key to the right place on macOS

Assume your downloaded key is `~/Downloads/my-ec2-key.pem`.

Move it into `~/.ssh/`:
```bash
mkdir -p ~/.ssh
mv ~/Downloads/my-ec2-key.pem ~/.ssh/
```

Set permissions (important for SSH):
```bash
chmod 400 ~/.ssh/my-ec2-key.pem
```

---

## 5) Get the EC2 public IP address

In AWS Console:
- EC2 → Instances → select your instance
- Look for **Public IPv4 address**
- Copy it

(Alternative: AWS CLI can fetch it, but the console is simplest.)

---

## 6) SSH into the instance from macOS

### Step 6.1 — SSH command template
Use:
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

### Step 6.2 — First-time connection prompt
If you see something like “Are you sure you want to continue connecting?”, type:
- `yes`

---

## 7) Common SSH problems (and fixes)

### Problem A — “Permission denied (publickey)”
Fix checklist:
- Key permissions too open:
  ```bash
  chmod 400 ~/.ssh/my-ec2-key.pem
  ```
- Correct username? (check `ec2-user` vs `ubuntu`)
- You’re using the right key pair for that instance

### Problem B — “Connection timed out” / “No route to host”
Usually means security group / networking issue:
- In EC2 **Security Group**, confirm inbound SSH (port 22)
- Confirm **Source** is your current public IP (or use a narrower range)

### Problem C — “Host key verification failed”
- This can happen if the instance was recreated and the IP now maps to a different host.
- You’ll need to remove the old known_hosts entry:
  ```bash
  ssh-keygen -R YOUR_PUBLIC_IP
  ```
- Then try SSH again.

---

## 8) Useful Linux command-line tools (inside the EC2 instance)

Once you’re connected, you may want basic tools. Names vary by distro:

### Amazon Linux (yum/dnf)
- `sudo yum update -y`
- `sudo yum install -y curl wget vim htop`

### Ubuntu/Debian (apt)
- `sudo apt update -y`
- `sudo apt install -y curl wget vim htop`

Useful basics to know:
- `whoami` (check your user)
- `uname -a` (kernel)
- `ip a` (network interfaces)
- `df -h` (disk)
- `free -h` (memory)
- `systemctl status <service>` (services, on systemd-based systems)

---

## 9) Optional: Use AWS CLI (find your public IP by instance-id)

If you know your `INSTANCE_ID`:
```bash
aws ec2 describe-instances \
  --instance-ids INSTANCE_ID \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text
```

---

## 10) Quick reference (copy/paste)

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
- Keep the `.pem` file private and never commit it to Git.
- If you change the instance or recreate it, the SSH host identity may change; you might need to update `known_hosts`.

---

## 11) Preview / “test” this README locally (macOS)

A README isn’t an executable program, so “testing” usually means confirming:
- Markdown renders correctly (headings, code blocks, lists)
- Links work (if you have internet)
- Any commands in code blocks are copy/paste-able

### Option A — Preview in VS Code (recommended)
1. Open `README.md` in VS Code
2. Use **Markdown Preview**:
   - Press **Shift + Command + P**
   - Type: `Markdown: Open Preview`
   - Press Enter

### Option B — Convert to HTML and open in a browser (terminal)
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

### Option C — Validate shell commands in the code blocks (manual)
- Copy one command at a time and run it in your terminal.
- Be careful with commands that require AWS credentials (e.g., anything using `aws ...`):
  - don’t run them until you’ve configured AWS access properly.
# ec2-docker
