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
