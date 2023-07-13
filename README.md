<h1>Brute Forcing Active Directory with Metasploit</h1>

<h2>Intro</h2>
Funny story how this lab began. I was actually attempting to set up another lab with Active Directory but could not remember the admin login I created initially. I kept trying generic passwords and ones that I used in the past but none could log me in. Frustrated, I tried recovery mode to see if I can get to the command prompt but that required my admin login as well 🤦‍♂️. This situation got my wheels spinning. Can I crack my password? I began to look into this and finally got it with a little help which will be discussed later on. Follow along after setting up AD and Kali or just read through my process on how dangerous it is to use weak passwords. I detailed the process and tools that I used below. Check out my first AD project or Jason Makador's Youtube tutorial on how to create the homelab.

<h2>🔩 Utlities Used</h2>

- Windows Server 2019 💻 (target)
- Virtual Box 🧰
- Kali Purple 🐉 (attacker)

<h2>♻️ The Process</h2>

1. Deep thought: How can I crack my password? In Windows, the password hashes are stored in HKEY_LOCAL_MACHINE\SAM. SAM is the Windows Security Accounts Manager. For obvious security purposes, this database cannot be accessed unless you have system priveleges and HLK/SAM access. Without getting to deep into Windows internals, I am on the outside so I can't even get to this file to extract the hashes to run through tools like Hashcat. My next thought: why not try brute forcing it?

2. Planning: I am familiar with tools like John the Ripper and Hydra for password cracking. I also recalled Burpsuite can be used to brute force logins from my ethical hacking course. I kept those tools in mind but first I needed to put Kali and the Server 2019 machine on the same network.

3. Network: VirtualBox has a variety of options under the Network setting for each machine. Most if not all of my machines are set to NatNetwork which I created as a default. I also created a ChryberHomeLab Natnetwork recently as well. A NatNetwork is an internal network that accepts outbound connections. In VirtualBox, Click Tools < Network < NAT Networks < Create < Enter network name < Enter IPv4 address (mine is a /24 network to allow for 254 hosts) < Enable DHCP (unless you want to assign IP addresses to each machine yourself) < Apply. Note: This should not be the same IP address as your host machine. I recommend using 'ipconfig' on your host machine command line and confirm your ip address.
![Recording 2023-07-12 at 19 03 37](https://github.com/chryber/Brute-Forcing-Active-Directory-with-Metasploit/assets/121698544/4d721f81-3ecb-40ab-829e-e51567b27c16)

4. Set up: Moving AD and Kali into the same subnet. Click machine < Settings < Network < Attached to: NAT Network < Name: ChryberHomeLab (in my case).
![Recording 2023-07-12 at 19 18 10](https://github.com/chryber/Brute-Forcing-Active-Directory-with-Metasploit/assets/121698544/2c60faa1-3403-491a-b02f-88a9bfcf892f)

5. IP address locator: Now that the attack machine (Kali) and the target (Server 2019) are on on the same network, my next thought was how I can locate the IP address assigned to the server machine by DHCP. The first thing I did was confirm Kali's ip using `ifconfig` which was 192.168.3.5 I decided to use Nmap. In short, Nmap scans a specific ip or subnet for open ports, services and hosts. I used the help page to use the best flags for this. I chose `nmap -v -sn 192.168.3.0/24`. Why? -v for verbosity so I can see the addresses being scanned and -sn so ports are not scanned for each, all I need to know is what hosts are active on the network. 
![NmapscanforAD](https://github.com/chryber/Brute-Forcing-Active-Directory-with-Metasploit/assets/121698544/6a1490f9-e220-4b74-8ec1-4d9ddc5d4234)
After narrowing the active hosts to 2 (which is correct since I have a Win 10 machine also on the network), I switched up the scan to all port scan using `nmap -T4 -p- -A 192.168.3.1` and `nmap -T4 -p- -A 192.168.3.4`. Once I saw Kerberos, I knew I had the right ip. I briefly looked over the open ports and services.
![NmapscanADfound!](https://github.com/chryber/Brute-Forcing-Active-Directory-with-Metasploit/assets/121698544/ce54026d-2f26-4f2c-a2aa-24fdc51b54ac)

6. Research: Next, I focused on which tools/methods I could use to brute force. First, I thought about John the Ripper but I remembered that I would need a password file to run it against which I did not have. Next, I checked out Burp. I have used some of it's features such as intercepter but this time would require intruder as well. After digging into it for a bit I quickly realized this would not be the best method since it requires the capturing of login traffic which isn't possible because I am not logging in over the internet. 
![Burpintercept](https://github.com/chryber/Brute-Forcing-Active-Directory-with-Metasploit/assets/121698544/b24c4ca8-8956-4497-a2eb-7559e77ce97f)

7. Revelation: During this process, I came to the realization that the account I was attempting to login to was a standard non admin user. I only needed to select "Other User" and then enter Administrator and default weak password of 'Password1' 🤦‍♂️. I felt stupid but I did not want to cheat myself out of the opportunity to learn more so I continued as if I never logged at all. I continued researching ways to break in until I found the WINRM exploit via https://www.youtube.com/watch?v=cF02JaXTCwc. Shout out to Pwn and Defend.

8. Enumerating: Simple enough the exploit can be found in Metasploit so I launched `msfconsole` from the Kali command line. I also read up on WIN RM. In summary, Windos Remote Manager is a built-in remote management protocol that enables admins to communicate with hosts and manage servers that are not in their immedicate vicinity but are on the same network. First step was to use the auxiliary module `auxiliary/scanner/winrm/winrm_auth_methods` or `use 1` to find out if a WINRM service is listening on http(s) and enumerate the supported authentication methods.</br>Using this module required a few settings first (TCM's Ethical Hacking course knowledge kicking in). By using `options` I saw that I needed to set the domain name and rhosts. The rport and URI are to be kept default. Here I saw Kerberos and Negotiate (NTLM) protocol are supported.</br>
`set DOMAIN MYFIRSTDOMAIN`</br>
`set rhost <target ip>`</br>
`run`
![msfconsole_winrm](https://github.com/chryber/Brute-Forcing-Active-Directory-with-Metasploit/assets/121698544/848eb407-1617-470a-b949-594f5ee9e767)

9. Exploit: Next, it was time to find this password. I loaded up the the brute force scanner `auxiliary/scanner/winrm/winrm_login` via `use 3`. Again, I used `options` to see what was required and set already. For this step, I needed the domain, rhosts, password, pass_file and username (even though password, pass_file and username says 'no'). The exploit wouldn't run without all 5 because of the authentication. For password, I put a random password value in but input the actual username. As for pass_file, this requires a file with passwords on one line so I thought the john.lst wordlist I had in Kali would be just fine.</br>
`set DOMAIN MYFIRSTDOMAIN`</br>
`set rhosts <target ip>`</br>
`set PASSWORD pass` (not the correct password btw)</br>
`set PASS_FILE usr/share/wordlists/john.lst`</br>
`set USERNAME Administrator`
![Bruteforcefailed](https://github.com/chryber/Brute-Forcing-Active-Directory-with-Metasploit/assets/121698544/92a5bf72-8289-4813-9879-92a0dd3bb2a9)

10. Try Again: So brute forcing failed the first time after running through each password in the file. The password I set was in the file but unfortunately, one capital letter made all the difference (I checked). Maybe I need more wordlists or maybe it was something else. I will figure that out later.

11. I decided to create my own wordlist with the password I set to test the module regardless. I opened a new terminal window and navigates to the wordlists directory. Writing to file in this directory is not possible unless you have root permissions so I sudo'd into root and created the file `mousepad brutefile.txt`. I loaded up some generic passwords including the correct one and reset the pass_file option to `set PASS_FILE usr/share/wordlists/brutefile.txt`. SUCCESS!
![Bruteforcecomplete!](https://github.com/chryber/Brute-Forcing-Active-Directory-with-Metasploit/assets/121698544/3511cb4e-7a39-40c6-8b6a-c26c2d4a3d6c)


