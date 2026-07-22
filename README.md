# CyberLab-01

**OBJECTIVE:**

You are a SOC analyst, and your company has recently experienced a DoS attack.
After the incident was resolved, your manager asked you to develop a method to protect the company’s infrastructure from being attacked again.

**TASK:**

Write a Suricata rule that detects future DoS attempts.
Note: I will perform this task using Kali Linux Virtual Machine

**1- Get your Kali Linux Up-to-date:**

```bash
sudo apt update
sudo apt upgrade
```




**2- Installing Suricata:**

```bash
sudo apt install suricata -y
Note: (-y) means yes to all.
```

You can verify your Suricata version by typing (Suricata)
Press enter or click to view image in full size

You can find Suricata configuration files in the following path: (/etc/suricata)
Press enter or click to view image in full size

**3- Configuring Suricata:**

1- Take a note of your running IP address by (ip a).
Press enter or click to view image in full size
2- Now open the Suricata configuration file
sudo mousepad /etc/suricata/suricata.yaml
3- Edit the following in the configuration file:
Update the HOME_NET with the network range your are in (for me it will be 192.168.232.0/24) as noted before then save.

Now the Suricata is ready to work and waiting for us to make rules and see the magic but first lets update the modification we just did.

sudo suricata-update
Press enter or click to view image in full size

4- Making Ping Flood Rules:
Note: to write any rule in a file just make sure that it ends with extension (.rules) that’s the way Suricata understands it.

touch DoS.rules
sudo mousepad Dos.rules
Write the following rules:

alert icmp any any -> $HOME_NET any (msg:"[!] PING flood detection - excessive amount of echo requests"; itype:8; flow:to_server; threshold: type limit, track by_src, count 100, seconds 10; classtype:attempted-dos; sid:1000010;)

alert icmp any any -> $HOME_NET any ( msg:"[!] PING flood detection - rapid icmp echo requests"; itype:8; flow:to_server; detection_filter: track by_src, count 50, seconds 1; classtype:attempted-dos; sid:1000011;)
Press enter or click to view image in full size

Then save.

Explanation:
