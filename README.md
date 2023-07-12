<h1>Brute Forcing Active Directory with Metasploit</h1>

<h2>Intro</h2>
Funny story how this lab began. I was actually attempting to set up another lab with Active Directory but could not remember the admin login I created initially. I kept trying generic passwords and ones that I used in the past but none could log me in. Frustrated, I tried recovery mode to see if I can get to the command prompt but that required my admin login as well. 🤦‍♂️
This situation got my wheels spinning. Can I crack my AD password? I began to look into this and finally got it with a little help from . I detailed the process and tools that I used below. Check out my first AD project or Jason Makador's Youtube tutorial on how to create the homelab.

<h2>🔩 Utlities Used</h2>

- Windows Server 2019 💻
- Virtual Box 🧰
- Kali Purple 🐉

<h2>♻️ The Process</h2>

1. Deep thought: How can I crack my password? In Windows, the password hashes are stored in HKEY_LOCAL_MACHINE\SAM. SAM is the Windows Security Accounts Manager. For obvious security purposes, this database cannot be accessed unless you have system priveleges and HLK/SAM access. Without getting to deep into Windows internals, I am on the outside so I can't even get to this file to extract the hashes to run through tools like Hashcat. My next thought: why not try brute forcing it?

2. Planning: I am familiar with tools like John the Ripper and Hydra for password cracking. I also recalled Burpsuite can be used to brute force logins from my ethical hacking course. I kept those tools in mind but first I needed to put Kali and the Server 2019 machine on the same network.

3. Network: VirtualBox has a variety of options under the Network setting for each machine. Most if not all of my machines are set to NatNetwork which I created as a default. I also created a ChryberHomeLab Natnetwork recently as well. A NatNetwork is an internal network that accepts outbound connections.
        - In VirtualBox < Tools < Netowrk < NAT Networks < Enter network name < Enter IPv4 address (mine is a /24 network to allow for 254 hosts) < Enable DHCP (unless you want to assign IP addresses to each machine yourself) < Apply.
![Recording 2023-07-12 at 19 03 37](https://github.com/chryber/Brute-Forcing-Active-Directory-with-Metasploit/assets/121698544/4d721f81-3ecb-40ab-829e-e51567b27c16)
