# 🚀 WireGuard VPN on AWS EC2 - Complete Setup Guide

This guide will help you set up your own VPN server using WireGuard on AWS EC2. It's written in simple language - no deep technical knowledge required!

## 📋 Table of Contents

- [What You'll Need](#what-youll-need)
- [How WireGuard Keeps You Safe](#how-wireguard-keeps-you-safe)
- [Step-by-Step Setup](#step-by-step-setup)
- [Setting Up Your Devices](#setting-up-your-devices)
- [Adding More Devices](#adding-more-devices)
- [Using AdGuard DNS for Ad Blocking](#using-adguard-dns-for-ad-blocking)
- [Troubleshooting](#troubleshooting)
- [Scripts](#scripts)

## 🎯 What You'll Need

Before we start, make sure you have:

- An AWS account
- A running EC2 instance with Ubuntu (20.04 or newer)
- An Elastic IP attached to your instance (this gives you a permanent IP address)
- Your `.pem` key file for SSH access
- A computer with terminal/command line access

## 🔐 How WireGuard Keeps You Safe

**Good news!** WireGuard automatically encrypts all your internet traffic. Here's what you need to know:

- ✅ **Military-grade encryption** is built-in - no extra setup needed!
- ✅ Uses the **latest cryptography** (ChaCha20, Poly1305, Curve25519, BLAKE2s)
- ✅ Even more secure than traditional VPNs that use SSL
- ✅ Your ISP or anyone snooping on public WiFi **cannot see** your internet activity
- ✅ All traffic between your device and the VPN server is **completely encrypted**

Think of it like a secure tunnel - everything that goes through is scrambled so only you and your server can understand it.

## 🛠️ Step-by-Step Setup

### Step 1: Prepare Your AWS Instance

First, we need to configure AWS to allow VPN connections.

#### 1.1 Update Security Group

In your AWS Console:
1. Go to EC2 → Security Groups
2. Find your instance's security group
3. Add these inbound rules:
   - **SSH**: Port 22 (TCP) - for remote access
   - **WireGuard**: Port 51820 (UDP) - for VPN connections

#### 1.2 Disable Source/Destination Check (VERY IMPORTANT!)

This step is crucial - without it, your VPN won't route traffic properly:

1. Select your EC2 instance
2. Click Actions → Networking → Change source/destination check
3. **Uncheck** the box (disable it)
4. Save

> 💡 **Why?** AWS normally prevents instances from forwarding traffic. We need to disable this so your VPN can route your internet traffic.

### Step 2: Connect to Your Server

Open your terminal and connect to your EC2 instance:

```bash
ssh -i "your-key-file.pem" ubuntu@your-server-ip
```

Replace:
- `your-key-file.pem` with your actual key file
- `your-server-ip` with your Elastic IP address

### Step 3: Run the Setup Script

Once connected, we'll use a setup script to install WireGuard. First, create the script:

```bash
nano setup-wireguard.sh
```

Copy the setup script from the [Scripts section](#setup-script) below, paste it, then:

```bash
# Make it executable
chmod +x setup-wireguard.sh

# Run it
./setup-wireguard.sh
```

The script will ask for your server's public IP (Elastic IP) and then set everything up automatically!

### Step 4: Download Your VPN Configuration

After the setup completes, you need to get the configuration file to your device:

```bash
# Make the config accessible
sudo cp /etc/wireguard/client1.conf /home/ubuntu/
sudo chown ubuntu:ubuntu /home/ubuntu/client1.conf
```

Now, from your local computer (not the EC2 instance), download it:

```bash
scp -i "your-key-file.pem" ubuntu@your-server-ip:~/client1.conf ~/Desktop/
```

## 📱 Setting Up Your Devices

### On iPhone/iPad
1. Download WireGuard from the App Store
2. Open the app and tap "+"
3. Choose "Create from QR code" or "Create from file or archive"
4. Import your configuration file

### On Mac
1. Download WireGuard from [wireguard.com](https://www.wireguard.com/install/)
2. Open WireGuard
3. Click "Import tunnel(s) from file"
4. Select your downloaded `.conf` file
5. Click "Activate"

### On Windows
1. Download WireGuard from [wireguard.com](https://www.wireguard.com/install/)
2. Click "Add Tunnel" → "Import from file"
3. Select your `.conf` file
4. Click "Activate"

### On Android
1. Install WireGuard from Google Play Store
2. Tap the "+" button
3. Choose "Import from file"
4. Select your configuration file

## ➕ Adding More Devices

**Important Security Note:** Each device needs its own unique configuration. Never share configuration files between devices!

### Why Separate Configs?

- 🔐 **Security**: Each device has its own encryption keys
- 📊 **Monitoring**: You can see which devices are connected
- 🚫 **Access Control**: You can remove individual devices if needed
- 🐛 **Troubleshooting**: Easier to fix problems with specific devices

### Adding a New Device

1. Connect to your server via SSH
2. Run the add-client script:

```bash
./add-client.sh iPhone
./add-client.sh Laptop
./add-client.sh AndroidTablet
```

3. Download the new configuration file:

```bash
scp -i "your-key.pem" ubuntu@your-server-ip:~/iPhone.conf ~/Desktop/
```

## 🛡️ Using AdGuard DNS for Ad Blocking

Want to block ads and trackers automatically? You can use AdGuard DNS instead of Google's DNS!

### Option 1: Standard AdGuard DNS (Recommended)
Blocks ads and tracking:
```
DNS = 94.140.14.14, 94.140.15.15
```

### Option 2: AdGuard Family DNS
Blocks ads, tracking, and adult content:
```
DNS = 94.140.14.15, 94.140.15.16
```

### Option 3: AdGuard Non-filtering DNS
No blocking, just secure DNS:
```
DNS = 94.140.14.140, 94.140.15.141
```

To change DNS in existing configs, edit the configuration file and replace the DNS line with one of the options above.

## 🔧 Troubleshooting

### Can't Connect to Internet Through VPN?

1. **Check if WireGuard is running:**
   ```bash
   sudo wg show
   ```

2. **Verify IP forwarding is enabled:**
   ```bash
   sysctl net.ipv4.ip_forward
   ```
   Should show `1`

3. **Check your network interface:**
   ```bash
   ip route | grep default
   ```
   Make sure the interface matches what's in your config (usually `ens5`)

4. **Restart WireGuard:**
   ```bash
   sudo wg-quick down wg0
   sudo wg-quick up wg0
   ```

### Connection Drops Frequently?

Make sure `PersistentKeepalive = 25` is in your client configuration. This keeps the connection alive through firewalls.

### Can't Download Config File?

If you get permission denied:
```bash
# On server
sudo cp /etc/wireguard/client1.conf /home/ubuntu/
sudo chown ubuntu:ubuntu /home/ubuntu/client1.conf

# Then try downloading again
```

## 📜 Scripts

### Setup Script

Save this as `setup-wireguard.sh`:

```bash
#!/bin/bash

# WireGuard AWS EC2 Setup Script
# This script automates the entire WireGuard installation process

set -e  # Exit on any error

echo "🚀 WireGuard AWS EC2 Setup Script"
echo "================================="

# Get server IP
read -p "Enter your AWS Elastic IP: " SERVER_IP

echo "📦 Installing WireGuard..."
sudo apt update && sudo apt upgrade -y
sudo apt install wireguard wireguard-tools -y

echo "🔧 Configuring system..."
# Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf

# Detect network interface
INTERFACE=$(ip route | grep default | awk '{print $5}')
echo "✅ Detected network interface: $INTERFACE"

# Create WireGuard directory
sudo mkdir -p /etc/wireguard
cd /etc/wireguard

echo "🔐 Generating server keys..."
wg genkey | sudo tee server_private.key | wg pubkey | sudo tee server_public.key
sudo chmod 600 server_private.key

# Get keys
SERVER_PRIVATE_KEY=$(sudo cat server_private.key)
SERVER_PUBLIC_KEY=$(sudo cat server_public.key)

echo "📝 Creating server configuration..."
sudo tee /etc/wireguard/wg0.conf << EOF
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = ${SERVER_PRIVATE_KEY}
PostUp = iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o ${INTERFACE} -j MASQUERADE; iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT
PostDown = iptables -t nat -D POSTROUTING -s 10.0.0.0/24 -o ${INTERFACE} -j MASQUERADE; iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT
EOF

echo "👤 Creating first client configuration..."
# Generate client keys
wg genkey | sudo tee client1_private.key | wg pubkey | sudo tee client1_public.key
CLIENT_PRIVATE_KEY=$(sudo cat client1_private.key)
CLIENT_PUBLIC_KEY=$(sudo cat client1_public.key)

# Add client to server
echo -e "\n[Peer]\nPublicKey = ${CLIENT_PUBLIC_KEY}\nAllowedIPs = 10.0.0.2/32" | sudo tee -a /etc/wireguard/wg0.conf

# Create client config with AdGuard DNS
sudo tee /etc/wireguard/client1.conf << EOF
[Interface]
PrivateKey = ${CLIENT_PRIVATE_KEY}
Address = 10.0.0.2/32
DNS = 94.140.14.14, 94.140.15.15
MTU = 1420

[Peer]
PublicKey = ${SERVER_PUBLIC_KEY}
Endpoint = ${SERVER_IP}:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
EOF

echo "🚀 Starting WireGuard..."
sudo wg-quick up wg0
sudo systemctl enable wg-quick@wg0

echo "✅ Setup complete!"
echo ""
echo "📋 Next steps:"
echo "1. Make sure UDP port 51820 is open in your AWS security group"
echo "2. Make sure source/destination check is DISABLED in AWS"
echo "3. Download client config: scp ubuntu@${SERVER_IP}:~/client1.conf ."
echo ""
echo "📊 Server status:"
sudo wg show
```

### Add Client Script

Save this as `add-client.sh`:

```bash
#!/bin/bash

# WireGuard Add Client Script
# Usage: ./add-client.sh ClientName

if [ -z "$1" ]; then
    echo "❌ Error: Please provide a client name"
    echo "Usage: ./add-client.sh ClientName"
    echo "Example: ./add-client.sh iPhone"
    exit 1
fi

CLIENT_NAME=$1
SERVER_IP="YOUR_SERVER_IP"  # Change this to your server's IP

echo "➕ Adding new client: ${CLIENT_NAME}"

# Find next available IP
LAST_IP=$(sudo grep "AllowedIPs" /etc/wireguard/wg0.conf | tail -1 | grep -oE '10\.0\.0\.[0-9]+' | cut -d. -f4)
NEXT_IP=$((LAST_IP + 1))
CLIENT_IP="10.0.0.${NEXT_IP}"

echo "📍 Assigning IP: ${CLIENT_IP}"

# Generate keys
cd /etc/wireguard
wg genkey | sudo tee ${CLIENT_NAME}_private.key | wg pubkey | sudo tee ${CLIENT_NAME}_public.key
sudo chmod 600 ${CLIENT_NAME}_private.key

# Get keys
PRIVATE_KEY=$(sudo cat ${CLIENT_NAME}_private.key)
PUBLIC_KEY=$(sudo cat ${CLIENT_NAME}_public.key)
SERVER_PUBLIC=$(sudo cat server_public.key)

# Add to server config
echo "📝 Adding peer to server..."
echo -e "\n[Peer]\n# ${CLIENT_NAME}\nPublicKey = ${PUBLIC_KEY}\nAllowedIPs = ${CLIENT_IP}/32" | sudo tee -a /etc/wireguard/wg0.conf

# Create client config with AdGuard DNS
sudo tee /etc/wireguard/${CLIENT_NAME}.conf << EOF
[Interface]
PrivateKey = ${PRIVATE_KEY}
Address = ${CLIENT_IP}/32
DNS = 94.140.14.14, 94.140.15.15
MTU = 1420

[Peer]
PublicKey = ${SERVER_PUBLIC}
Endpoint = ${SERVER_IP}:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
EOF

# Restart WireGuard
echo "🔄 Restarting WireGuard..."
sudo wg-quick down wg0 && sudo wg-quick up wg0

# Make config downloadable
sudo cp /etc/wireguard/${CLIENT_NAME}.conf /home/ubuntu/
sudo chown ubuntu:ubuntu /home/ubuntu/${CLIENT_NAME}.conf

echo "✅ Client added successfully!"
echo ""
echo "📥 Download the config with:"
echo "scp ubuntu@${SERVER_IP}:~/${CLIENT_NAME}.conf ."
echo ""
echo "📊 Active connections:"
sudo wg show
```

### Management Script

Save this as `wg-manage.sh`:

```bash
#!/bin/bash

# WireGuard Management Script
# Simple menu for common WireGuard tasks

show_menu() {
    echo "🎛️  WireGuard Management Menu"
    echo "============================"
    echo "1) Show status"
    echo "2) Add new client"
    echo "3) List all clients"
    echo "4) Restart WireGuard"
    echo "5) Show server public key"
    echo "6) Exit"
}

while true; do
    show_menu
    read -p "Choose an option: " choice
    
    case $choice in
        1)
            echo "📊 WireGuard Status:"
            sudo wg show
            ;;
        2)
            read -p "Enter client name: " client_name
            ./add-client.sh "$client_name"
            ;;
        3)
            echo "👥 Configured clients:"
            sudo grep "# " /etc/wireguard/wg0.conf | sed 's/# /  - /'
            ;;
        4)
            echo "🔄 Restarting WireGuard..."
            sudo wg-quick down wg0 && sudo wg-quick up wg0
            echo "✅ Restarted"
            ;;
        5)
            echo "🔑 Server public key:"
            sudo cat /etc/wireguard/server_public.key
            ;;
        6)
            echo "👋 Goodbye!"
            exit 0
            ;;
        *)
            echo "❌ Invalid option"
            ;;
    esac
    
    echo ""
    read -p "Press Enter to continue..."
    clear
done
```

## 📁 Repository Structure

When you create your GitHub repository, organize it like this:

```
wireguard-aws-setup/
├── README.md           (this file)
├── scripts/
│   ├── setup-wireguard.sh
│   ├── add-client.sh
│   └── wg-manage.sh
└── docs/
    └── troubleshooting.md
```

## 🎉 Congratulations!

You now have your own private VPN server! Your internet traffic is encrypted and secure, and if you're using AdGuard DNS, you're also blocking ads and trackers.

## 📝 License

Feel free to use and modify these scripts for your own needs!

---

Made with ❤️ for privacy and security enthusiasts