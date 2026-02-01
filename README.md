# OpenClaw Setup Guide: AWS EC2 (Amazon Linux 2023) with Claude Opus 4.5

> **DISCLAIMER OF WARRANTIES AND LIMITATION OF LIABILITY**
> 
> **USE AT YOUR OWN RISK.** This guide and the OpenClaw software are provided "AS IS" without warranty of any kind, express or implied, including but not limited to warranties of merchantability, fitness for a particular purpose, or non-infringement.
> 
> **NO LIABILITY:** The author(s) and contributors of this guide assume no responsibility or liability for any damages, losses, costs, or expenses (including but not limited to direct, indirect, incidental, special, consequential, or punitive damages) arising from:
> - Use or misuse of OpenClaw or this guide
> - Security vulnerabilities, breaches, or unauthorized access
> - Data loss, corruption, or exposure
> - API costs incurred through Anthropic or other services
> - AWS EC2 costs or charges
> - System failures, downtime, or service interruptions
> - Any actions taken by OpenClaw or AI models
> - Incorrect configuration or implementation
> 
> **SECURITY RISKS:** OpenClaw has full system access and can execute commands, access files, and make network requests. There are inherent security risks in using such software. You are solely responsible for:
> - Securing your systems and data
> - Managing API keys and credentials
> - Setting appropriate spending limits
> - Monitoring usage and activity
> - Complying with applicable laws and regulations
> - Backing up your data
> 
> **NO SUPPORT:** This guide is provided for informational purposes only. No support, maintenance, or updates are guaranteed or implied.
> 
> By using this guide or OpenClaw, you acknowledge that you have read, understood, and agree to assume all risks and responsibilities associated with its use.

## Overview

This guide covers deploying OpenClaw on **AWS EC2 Free Tier** using **Claude Opus 4.5** via SSH installation. EC2 provides complete isolation from your local machine, eliminating security risks while providing 12 months of free hosting.

**Why EC2?**
- ✅ Complete isolation from local network
- ✅ Free for 12 months (750 hours/month of t3.micro)
- ✅ 24/7 availability
- ✅ Easy to destroy/recreate for experimentation

**⚠️ Important: t3.micro Limitations**
- Only **1GB RAM** - tight for npm installs
- **REQUIRED:** Add swap space (Step 3) or installation will freeze
- **REQUIRED:** Increase Node.js memory limit (Step 4) or you'll get "heap out of memory" errors
- Installation takes **15-20 minutes** on t3.micro
- **Alternative:** Use t3.small (2GB RAM) for easier installation - costs ~$15/month after free tier

**Why Claude Opus 4.5?**
- Most capable model for complex reasoning and code generation
- 200K context window
- API pricing: $5/MTok input, $25/MTok output

## Prerequisites

- AWS account (new accounts get 12 months free tier)
- Anthropic API key with Claude Opus 4.5 access
- Terminal/SSH access
- Budget for API costs (Opus 4.5 is premium pricing)

**Estimated Costs:**
- **EC2 Hosting:** Free for 12 months, then ~$8/month
- **API Costs (Opus 4.5):**
  - Light usage: ~$150-300/month
  - Moderate usage: ~$500-1,000/month
  - Heavy usage: $2,000-9,000/month
- **Set spending limits** in Anthropic Console before starting

## Step 1: Launch EC2 Instance

1. **Log into AWS Console**
   - Go to: https://console.aws.amazon.com/ec2/
   - Select your preferred region

2. **Launch Instance**
   - Click "Launch Instance"
   - Name: `openclaw`

3. **Choose AMI**
   - **Amazon Linux 2023**
   - Architecture: **64-bit (x86)** (better Node.js compatibility)
   - Kernel version: Doesn't matter (6.1 or 6.12 both work)

4. **Instance Type**
   - **t3.micro** (Free Tier eligible)
   - 2 vCPU, 1 GiB Memory
   - Free for 12 months (750 hours/month), then ~$7.50/month

5. **Key Pair**
   - Create new key pair
   - Name: `openclaw-key`
   - Type: **RSA** (recommended)
   - Format: **.pem**
   - **⚠️ CRITICAL:** Download and save the .pem file securely - you cannot download it again
   - Set permissions: `chmod 400 openclaw-key.pem`

6. **Network Settings**
   - Create new security group
   - **SSH (22):** Allow from "My IP" only
   - **Custom TCP (18789):** Allow from "My IP" (for dashboard)

7. **Storage**
   - 8GB gp3 (Free Tier includes 30GB)

8. **Launch**
   - Review settings and launch
   - Wait for instance to be "Running"
   - Copy the **Public IPv4 address**

## Step 2: Connect via SSH

```bash
ssh -i openclaw-key.pem ec2-user@YOUR_PUBLIC_IP
```

- Type `yes` when prompted about host authenticity
- You should see Amazon Linux welcome message

**Verify curl is installed:**
```bash
curl --version
```

If not installed: `sudo yum install -y curl` (usually pre-installed)

## Step 3: Add Swap Space (Required)

**⚠️ Critical:** t3.micro has only 1GB RAM. Without swap space, npm installs will freeze.

```bash
# Create 2GB swap file
sudo dd if=/dev/zero of=/swapfile bs=1M count=2048
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make it permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Verify swap is active
free -h
```

You should see 2GB swap listed in the output.

## Step 4: Increase Node.js Memory Limit (Required)

**⚠️ Critical:** Node.js default heap limit (~512MB) will cause "JavaScript heap out of memory" errors. Set this BEFORE running the installer.

**Option 1: Set environment variable (Recommended):**
```bash
export NODE_OPTIONS="--max-old-space-size=2048"
echo 'export NODE_OPTIONS="--max-old-space-size=2048"' >> ~/.bashrc
source ~/.bashrc
echo $NODE_OPTIONS  # Verify it's set
```

**Option 2: Set inline when running installer:**
```bash
NODE_OPTIONS="--max-old-space-size=2048" curl -fsSL https://clawd.bot/install.sh | bash
```

## Step 5: Install OpenClaw

```bash
curl -fsSL https://clawd.bot/install.sh | bash
```

**What this does:**
- Installs Node.js (if needed)
- Installs OpenClaw globally
- Sets up systemd service for 24/7 operation

**Installation takes:** ~5-15 minutes on t3.micro

**Verify installation:**
```bash
openclaw --version
sudo systemctl status openclaw
```

## Step 6: Configure with Claude Opus 4.5

```bash
openclaw onboard --cloud
```

**During onboarding:**
1. Select **Anthropic** as API provider
2. Enter your Anthropic API key (get from https://console.anthropic.com/settings/keys)
3. Select **`claude-opus-4-5`** or `claude-opus-4.5`
4. Configure communication channels (WhatsApp/Telegram/Discord)
5. Confirm cloud deployment mode

**Verify configuration:**
```bash
openclaw config show
openclaw test
openclaw gateway status
```

## Step 7: Set Up API Spending Limits

**Critical:** Opus 4.5 is expensive. Set limits before using.

1. Go to: https://console.anthropic.com/settings/limits
2. Set daily limit: **$10-20/day** (start conservative)
3. Set monthly limit: **$300-500/month**
4. Enable email alerts at 50%, 80%, 100%

**Monitor usage:**
- Web: https://console.anthropic.com/usage
- API: `curl https://api.anthropic.com/v1/usage -H "x-api-key: YOUR_KEY" -H "anthropic-version: 2023-06-01"`

## Step 8: Access Dashboard

1. **Configure Security Group:**
   - EC2 → Security Groups → Select your instance's security group
   - Edit Inbound Rules → Add Custom TCP Port 18789 from "My IP"

2. **Access Dashboard:**
   ```
   http://YOUR_PUBLIC_IP:18789
   ```

## Step 9: Set Up Communication Channels

**WhatsApp:**
```bash
openclaw gateway whatsapp
# Scan QR code with WhatsApp
```

**Telegram:**
1. Message @BotFather → `/newbot` → Copy token
2. `openclaw gateway telegram` → Enter token

**Discord:**
1. https://discord.com/developers/applications → Create bot → Copy token
2. `openclaw gateway discord` → Enter token

## Step 10: Test OpenClaw

Send a test message via your configured channel:
```
Hello! What can you do?
```

**Test Opus 4.5 capabilities:**
- Complex reasoning tasks
- Code analysis and generation
- Long-form content creation
- Multi-step problem solving

## Security Considerations

**⚠️ Important:** OpenClaw has full system access and can execute commands, access files, and make network requests. Consider these risks:

**Current Protections:**
- ✅ EC2 isolated from local network
- ✅ SSH restricted to your IP
- ✅ Dashboard port restricted

**Best Practices:**
- Use separate API keys (not production)
- Don't connect banking/financial services initially
- Monitor API usage for unexpected activity
- Rotate API keys regularly
- Review OpenClaw actions regularly
- Start with read-only access where possible

**Additional Security:**
- Enable AWS CloudWatch monitoring
- Regular system updates: `sudo yum update -y`
- Backup configuration: `openclaw config export > backup.json`

## Cost Optimization

**Model Selection:**
- **Opus 4.5:** Complex reasoning, code generation ($5/$25 per MTok)
- **Sonnet 4.5:** General automation ($3/$15 per MTok - 60% cheaper)
- **Haiku 4.5:** Simple tasks ($1/$5 per MTok - 80% cheaper)

**Optimization Strategies:**
- Use Opus for complex tasks only
- Use Sonnet/Haiku for simple operations
- Enable prompt caching for repeated operations
- Use batch processing for non-urgent tasks
- Monitor usage patterns and optimize

## Troubleshooting

**Can't connect via SSH:**
- Check security group allows your IP
- Verify key file permissions (`chmod 400`)
- Verify username is `ec2-user` (not `ubuntu`)

**"JavaScript heap out of memory" error:**
- Set `NODE_OPTIONS="--max-old-space-size=2048"` before installer
- Or run: `NODE_OPTIONS="--max-old-space-size=2048" curl -fsSL https://clawd.bot/install.sh | bash`

**Installation freezes:**
- Verify swap space is active: `free -h`
- Verify Node.js memory limit: `echo $NODE_OPTIONS`
- Wait 15-20 minutes (npm installs are slow on t3.micro)
- Check if process is running: `ps aux | grep npm`

**OpenClaw not responding:**
```bash
sudo systemctl status openclaw
sudo systemctl restart openclaw
sudo journalctl -u openclaw -f
```

**Dashboard not accessible:**
- Check security group allows port 18789
- Verify instance is running

## Cost Estimates (Claude Opus 4.5)

**Per Million Tokens:**
- Input: $5 / MTok
- Output: $25 / MTok

**Example Monthly Costs:**
- Light usage (100 interactions/day): ~$180/month
- Moderate usage (500 interactions/day): ~$1,350/month
- Heavy usage (2,000+ interactions/day): ~$9,000/month

**⚠️ Important:** Start with conservative limits and monitor closely.

## Resources

- **OpenClaw Docs:** https://openclaw.ai/
- **OpenClaw GitHub:** https://github.com/openclaw/openclaw
- **AWS EC2 Free Tier:** https://aws.amazon.com/free/
- **Anthropic API Docs:** https://docs.anthropic.com/
- **Anthropic Pricing:** https://docs.anthropic.com/en/docs/about-claude/pricing


