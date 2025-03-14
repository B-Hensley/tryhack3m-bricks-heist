# TryHack3M: Bricks Heist - Technical Write-Up

This repository contains the technical write-up for the TryHackMe TryHack3M: Bricks Heist room. The write-up outlines the steps taken to complete the room, detailing the reconnaissance, exploitation, and investigation processes involved in the capture-the-flag challenge.

## Table of Contents

1. [Recon & Initial Investigation](#recon--initial-investigation)
2. [Exploitation](#exploitation)
3. [Exploring Initial Access](#exploring-initial-access)
4. [Identifying the Suspicious Process](#identifying-the-suspicious-process)
5. [Investigating the BTC Address](#investigating-the-btc-address)
6. [Conclusion](#conclusion)

## Recon & Initial Investigation

Upon entering the TryHackMe: Bricks Heist room, I started both the AttackBox and the victim system. After adding the victim's IP to the `/etc/hosts` file, I investigated the webpage at `https://bricks.thm`, confirming it was a WordPress site. I ran a WPScan to identify the WordPress version (1.9.5), which was vulnerable to CVE-2024-25600, leading to the discovery of an exploit on GitHub.

## Exploitation

I downloaded the raw exploit code for the Bricks WordPress vulnerability and ran it using Python. After installing required Python modules (`rich`, `prompt-toolkit`, and `alive-progress`), the exploit successfully ran, confirming the vulnerability was present.

## Exploring Initial Access

After exploiting the vulnerability, I accessed the victim system with Apache privileges. I used a reverse shell to stabilize the connection and discovered a flag in a `.txt` file. Additionally, I identified hard-coded credentials in the `wp-config.php` file, which could potentially be used for further exploitation.

## Identifying the Suspicious Process

Upon checking the system’s running processes, I identified a suspicious `ubuntu.service` running from the `/lib/NetworkManager/nm-inet-dialog` directory. Investigating the `inet.conf` file, I discovered that it contained logs related to a Bitcoin Miner. This led to identifying the Bitcoin mining operation running on the system.

## Investigating the BTC Address

By decoding an ID found in the Bitcoin Miner logs, I traced the address to a Bitcoin wallet, which was confirmed to be associated with a Bitcoin Miner. Using a website to check the wallet's status, I discovered that the wallet had received and sent BTC and was linked to the LockBit ransomware.

## Conclusion

Through careful reconnaissance, exploitation, and analysis of the system and its processes, I successfully discovered the Bitcoin wallet address and linked it to the LockBit ransomware, completing the TryHackMe: Bricks Heist room.

