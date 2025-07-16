# Linux Log Analysis Lab - John Yang

## Objective

Today, I decided to try a SOC Analyst lab focused on simulating a real-world incident response scenario where I investigate suspicious activity on a compromised webserver honeypot by analyzing logs through Linux. I will be documenting my thought process and the actions that I took to navigate through the questions asked. My primary objective for this lab was to practice and brush up my skills in smoothly navigating through Linux's CLI and to further develop my attention to detail in endpoint forensics and log analysis.

### Skills Learned

- Performed endpoint forensics and in‑depth log analysis to uncover attackers' various tactics, techniques, and procedures (TTPs).
- Developed fluency in Linux command‑line navigation and file system management to efficiently locate and analyze key information, streamlining SOC operations.
- Gained extensive knowledge of the unique contents of different logs in Linux, such as auth logs, Apache access logs, and package manager logs.
- Applied open‑source intelligence (OSINT) techniques to gather and correlate external data for threat analysis.

### Tools Used

- Windows Subsystem for Linux (WSL, Ubuntu distribution)
- Linux command-line commands & tools
- Open-source intelligence (OSINT)
  
## Before we begin

Navigate to https://cyberdefenders.org/blueteam-ctf-challenges/hammered/, download the lab zip file and extract it on your device. Then, open the lab folder in the terminal.

![alt text](OpenInTerminal.jpg)

Lets log in! We should be in the same directory as we were before.

![alt text](wsllogin.jpg)

Note that I used Windows Subsystem for Linux for this lab, but it is completely possible to do this with a virtual machine too.

Let's run ls -la to see all the information in our directory.

![alt text](DirectoryLS.jpg)

Now we're ready to begin!

## Questions

### Question 1: Which service did the attackers use to gain access to the system?

When dealing with attackers gaining access, it is most likely that they used valid credentials to bypass authentication. 

We can cat the auth.log file, and also we can pipe it with a grep to search for the word "accepted" to try to seek out successful authentications.

![alt text](authlog.jpg)

We can see sshd being used to gain access to the system.

Answer: **SSH**

### Question 2: What is the operating system version of the targeted system?

Going back to our directory and seeing its contents, we know that messages can contain information about system related events.

This time, instead of cat, I will use head to only output the first ten lines.

![alt text](messages.jpg)

And there it is! We see the operating system that the targeted system is running.

Answer: **4.2.4-1ubuntu3**

### Question 3: What is the name of the compromised account?

To tackle this question we would need to go back to auth.log.

I used the same command from before, `cat auth.log | grep -i accepted`, but I just wanted to clear up some things on the output, so I used another pipe to chain `awk '{print $1, $2, $9, $11}'`

This only shows the information from the columns associated with the numbers, so I narrowed down the output to the month, date, user, and IP address, which I determined were key information.

![alt text](FirstExternalIP.jpg)

Looking at the output, we can see that our first successful authentication from an external IP address was from 76.191.195.140 on the user1 account. We'll keep note of that.

Scrolling down further however, we see some more interesting information.

![alt text](RootExternalIP.jpg)

Here, we can see that our root account was accessed by a private IP on March 29. However, on April 19, we see that the account was accessed by an external IP address of 219.150.161.20.

To determine which account was the compromised one, I used some OSINT to find the geolocation of the two IP addresses in question.

![alt text](SantaRosa.jpg)

![alt text](Shanghai.jpg)

So now we have the locations of the two IP addresses, but I made sure to not make quick judgements and investigate further.

Digging around in different logs, I viewed user.log, where the timezone of the device was set to America/Los Angeles, or Pacfic Standard Time.

![alt text](messages.jpg)

Based on this information, we can finally deduct that the IP address from Shanghai was the more malicious one rather than the IP from Santa Rosa (a part of the same timezone), and that the compromised account was the root account.

Answer: **root**

### Question 4: How many attackers, represented by unique IP addresses, were able to successfully access the system?

We can assume that the account that we are dealing with is root, since we identified it as the compromised account from the last question.

Again, we grep the auth.log for accepted, but I narrowed the output down further by only asking for the IP address column.

![alt text](GainedAccessIPs.jpg)

From here, we can simply take the information from last time and only output the results that are unique.

![alt text](UniqueRootIPs.jpg)

We don't want to count all of those individually, lets just add a `wc -l` to our command.

![alt text](wc.jpg)

We can see that the root account has been successfully accessed 18 times.

Answer: **18**

### Question 5: Which attacker's IP address successfully logged into the system the most number of times?

To figure this out, we start from what we previously had, but bundle up and count the IP addresses instead. We can use `uniq -c` for this, and further sort the occurences from greatest to least by piping the command with `sort -nr`.

![alt text](MostFrequentIP.jpg)

We see that the same IP address from Shanghai, 219.150.161.20, has successfully accessed root the most amount of times.

Answer: **219.150.161.20**

### Question 6: How many requests were sent to the Apache Server?

Here, we want to cd over to our apache2 directory, then cat www-access.log, since that is where requests are logged.

![alt text](apache2.jpg)

From here, we can simply count up all of the results from the output that we have.

![alt text](apache2count.jpg)

We see that the server has been requested 365 times.

Answer: **365**

### Question 7: How many rules have been added to the firewall?

I was not entirely sure where to find specific information about firewall rules in logs, but I knew that firewall rules can be configured using a command called iptables.

From this, I decided to grep for iptables on the entire directory using `*` at the end to search every log file and `-r` to recursively search so that other directories can be searched too.

![alt text](iptables.jpg)

We can see that a total of 6 firewall rules have been added to the firewall.

Answer: **6**

### Question 8: One of the downloaded files on the target system is a scanning tool. What is the name of the tool?

We can see all the packages that are installed on dpkg.log.

Lets cat the log file and see if anything is suspicious.

![alt text](dpkg.jpg)

Scroling through the output, we find nmap!

![alt text](nmap.jpg)

Answer: **nmap**

### Question 9: When was the last login from the attacker with IP 219.150.161.20? Format: MM/DD/YYYY HH:MM:SS AM

To answer this question, we need to go back to auth.log.

We can grep for "accepted" again, but this time, also grep the specific IP address 219.150.161.20.

![alt text](lastdate.jpg)

We see that the last login from the attacker was on Apr 19 05:56:05. 












## Conclusion

Coming into this project, my knowledge of EDR and SOAR technologies was largely theoretical, coming from studying for courses and certifications. Actually getting to implement these systems, connect them through APIs, and design a complete workflow from detection to response was both eye-opening and incredibly rewarding. I was able to bridge the gap between abstract concepts and real-world application, and acquire a deeper appreciation for how modern security operations function. This project bolstered my enthusiasm for cybersecurity and desire to keep learning through hands-on experiences, and I definitely will be doing more projects like these. Thank you for reading!

## Contact

Email: <johnyang4406@gmail.com>, <john_s_yang@brown.edu>

LinkedIn: <https://www.linkedin.com/in/john-yang-747726292/>
