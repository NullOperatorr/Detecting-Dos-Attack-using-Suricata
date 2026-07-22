# CyberLab-01

**OBJECTIVE:**

You are a SOC analyst, and your company has recently experienced a DoS attack.
After the incident was resolved, your manager asked you to develop a method to protect the company’s infrastructure from being attacked again.

**TASK:**

Write a Suricata rule that detects future DoS attempts.

Note: I will perform this task using Kali Linux Virtual Machine

---

**1- Get your Kali Linux Up-to-date:**

```bash
sudo apt update
sudo apt upgrade
```

---


**2- Installing Suricata:**

```bash
sudo apt install suricata -y
Note: (-y) means yes to all.
```

You can verify your Suricata version by typing (Suricata)

<img width="640" height="178" alt="1_6F2YMDI0Wi-HAFCY2GjwPw" src="https://github.com/user-attachments/assets/43ff64e1-b8fa-43a2-a1f5-453c1d265773" />


You can find Suricata configuration files in the following path: (/etc/suricata)

<img width="640" height="73" alt="1_CzKvc-qE9uSxHfAb4iihOQ" src="https://github.com/user-attachments/assets/f2db08c8-a5c1-4937-8f58-51add0c548eb" />

---

**3- Configuring Suricata:**

1- Take a note of your running IP address by (ip a).

<img width="640" height="177" alt="1_AG4wHHqb-yRq5jqmOKQxPA" src="https://github.com/user-attachments/assets/5199ef15-dcd1-4262-ba2d-c2688008295d" />


2- Now open the Suricata configuration file
```bash
sudo mousepad /etc/suricata/suricata.yaml
```

3- Edit the following in the configuration file:
Update the HOME_NET with the network range your are in (for me it will be 192.168.232.0/24) as noted before then save.

<img width="391" height="118" alt="1_8igNgkL9w6nnlH39XzqacA" src="https://github.com/user-attachments/assets/245402e4-5b4c-4e18-bf37-a7c5fc877677" />


Now the Suricata is ready to work and waiting for us to make rules and see the magic but first lets update the modification we just did.
```bash
sudo suricata-update
````
<img width="640" height="544" alt="1_gzMzWgBuL4CAja7gBMyuFw" src="https://github.com/user-attachments/assets/5bd02805-e266-438d-a950-2fde2cb9052c" />


----

***4- Making Ping Flood Rules:****

Note: to write any rule in a file just make sure that it ends with extension (.rules) that’s the way Suricata understands it.

```bash
touch DoS.rules
sudo mousepad Dos.rules
```
Write the following rules:

```bash
alert icmp any any -> $HOME_NET any (msg:"[!] PING flood detection - excessive amount of echo requests"; itype:8; flow:to_server; threshold: type limit, track by_src, count 100, seconds 10; classtype:attempted-dos; sid:1000010;)

alert icmp any any -> $HOME_NET any ( msg:"[!] PING flood detection - rapid icmp echo requests"; itype:8; flow:to_server; detection_filter: track by_src, count 50, seconds 1; classtype:attempted-dos; sid:1000011;)
```
<img width="640" height="129" alt="1_PUD1_tpNdnnpVBj1-N8w4w" src="https://github.com/user-attachments/assets/9265e0c1-a000-42ee-8d27-d42074431d0d" />

Then save.

***Explanation:***

*Rule 1:*

1- (alert icmp any any -> $HOME_NET any): any ICMP into local network.  
2- (itype:8) : means Echo requests.  
3- (flow:to_server): means going to server from a client.  
4- (threshold: type limit, track by_src, count 100, seconds 10): set a limit to number of messages received to alert.  
5- (classtype:attempted-dos): goes into this category.  

*Rule 2:*

1- (detection_filter: track by_src, count 50, seconds 1): detects and alert after 50 request in a second.

Note: both rule 1&2 are close to each other but they differ in the detection mode (Rule-1) detects 100req./10 sec (less sensitive & slow)
(Rule-2) detects 50req./1 sec (More sensitive & rapid)

---

**5- Edit in Suricata configuration file:**

1- Copy the path of the Dos.rules we created before (/home/Desktop/Dos.rules)

2- Open the Suricata configuration file

```bash
sudo mousepad /etc/suricata/suricata.yaml
```

and go down until you find the rule-files then paste the path of your rule and save.

<img width="437" height="158" alt="1_pFa-01xHIF28wF5dZHrPNg" src="https://github.com/user-attachments/assets/bc12cf8a-5c2f-4ee2-a594-85207d6e5223" />

3- Now update and test your rule and verify that there is no error:

```bash
sudo suricata -T -c /etc/suricata/suricata.yaml -i eth0
Note that: the (-T) means testing.
```
<img width="640" height="137" alt="image" src="https://github.com/user-attachments/assets/7b0ba0e3-4cee-4691-b620-f24de75d4dff" />

4- Lets start the Suricata

```bash
sudo suricata -c /etc/suricata/suricata.yaml -i eth0
```

<img width="640" height="101" alt="image" src="https://github.com/user-attachments/assets/357f20e6-9511-426a-b701-a636d98e0da9" />

Now we are ready to go, but lets make a small python code to test our rules and finish our task properly as requested from the manager.

---

**6- Python Ping-flood script:**

Now lets attack ourselves
This script will simulate a Dos Attack by ICMP Flooding which overwhelms the network by sending a large number of ping requests until making the target inaccessible. (in this case our Kali VM is the target)

```bash
# pip install python-ping
from pythonping import ping

# Get ip address or hostname
host_address = input("Enter single IP address or hostname: ")
while True:
    try:
        # Ping host
        result = ping(
            host_address,
            count=10000,
            size=1000,
            timeout=1
        )

        print(result)

    except KeyboardInterrupt:
        print("\n Ping flood stopped by user.")
        break

    except Exception as e:
        print(f"\n Error: {e}")
        break

input("\n Press Enter to continue . . .")
```

Run this script on your local PC and write the IP of the target (Kali VM).

<img width="542" height="868" alt="image" src="https://github.com/user-attachments/assets/e9796ed6-6890-4e7c-9e02-e3811ef3cd36" />
<img width="420" height="237" alt="image" src="https://github.com/user-attachments/assets/de0e9375-0134-483e-8afd-b3a1fa33bedf" />

---

**7- Checking Suricata log file:**
the path where Suricata log its detections is (/var/log/suricata/fast.log), lets take a look at it:

```bash
tail /var/log/suricata/fast.log
```

<img width="640" height="196" alt="image" src="https://github.com/user-attachments/assets/c2236b17-2d2f-42b4-9612-59f7b8104b68" />

Now its verified that our Suricata rules is working, detecting malicious actions and task is accomplished.

---

**8- Lessons Learned:**



