### Reconnaisance & Analysis Lab

### Objective

The primary objective of this project is to conduct a comprehensive vulnerability assessment by selecting, installing, and mastering designated industry-standard vulnerability analysis tools. This will involve setting up a controlled testing environment by installing a target operating system and associated services (e.g., Mail server) within a virtual machine, intentionally configured without advanced security hardening. The project will culminate in the practical application of the tool to scan a colleague’s system, followed by the synthesis of findings into a concise report detailing the discovered vulnerabilities, associated risks, and key insights gained from the assessment process.

### Skills Learned

- Network Reconnaissance & Scanning
- Vulnerability Assessment
- Authentication Testing
- Analysis & Reporting

### Tools Used

- Nmap
- Hydra
- OpenVAS

## Step 1 : Connectivity

Target server was accessed and a ping was initiated to the Kali Linux
machine so that the connectivity could be verified as working

<img width="520" height="212" alt="1" src="https://github.com/user-attachments/assets/ae766726-5259-4b15-9799-2cce215c0bb1" />

## Step 2: Reconnaissance Attack Using Nmap

In this case we already knew the IP address of the target device, but if we didn’t know this prior
to the attack, and the device was configured insecurely to respond to ICMP request packets, then
we can use NMAP to scan the local area network for devices and see which ones respond

<img width="656" height="193" alt="2" src="https://github.com/user-attachments/assets/5be3bb45-644c-47a5-9f1c-7a790a045252" />


Used the command `nmap -O -sV $SERVER_IP` to probe for information about the operating
system, services running, open ports and more on our target server

<img width="479" height="220" alt="O and sV" src="https://github.com/user-attachments/assets/daeaeef9-14c5-4355-9f85-2f1885b2a464" />

## Results: Open Ports
- TCP 22 :  the Secure Shell protocol. Specifically in this case, we can see that the SSH server in use by the target system is OpenSSH 9.2p1, running on a Debian Linux system.
  
- TCP 25 :  SMTP protocol, designed to send and receive emails to and from other SMTP servers. All communication over this protocol is done without encryption. The server has been configured to use the Postfix smtpd program for its email functions. SMTP servers are often abused for fishing and for spam if not configured correctly.

- TCP 80 : HTTP protocol, which is designed to allow hosting of website files and access by web browsers. The NGINX web server is being ran on the target host. HTTP traffic between server and clients is plaintext and completely unencrypted, so maybe we can use that to our advantage and record some credentials when a client tries to use the website via MITM attack.

- TCP 110 : POP3 protocol, which is designed to allow mail client software, such as Mozilla Thunderbird or Microsoft Outlook, to retrieve and send emails to the local email server. Pop3 on port 110 is plaintext, which means any emails or credentials shared between the local client and email server can be viewed easily if the traffic is interceptedby a malicious actor. The server is running the dovecot software in order to provide a point for client connections to the email server.
  
- TCP 143 : IMAP protocol, which is designed for a similar use as the above mentioned pop3 protocol. Imap on port TCP 143 also does not implement any kind of encryption, and as such it is vulnerable to MiTM attacks.

- TCP 443 : HTTPS protocol, a more secure version of the HTTP protocol which adopts and integrates the use of TLS certificates issued by trusted Certificate Authorities in order to verify that the owner of the domain is who they claim to be, as well as asymmetric key encryption between the HTTPS client and HTTPS server. Content shared between the client and server cannot be easily read if captured by a threat actor, and as such this would make an attack more difficult. 

- TCP 465 : SMTPS protocol, a more secure versionof the SMTP port that implements asymmetric key encryption and TLS certificates to validate the email server is who they say they are. This is a legacy port which should no longer be used for SMTP communication.
  
- TCP 587 :  SMTPS protocol, a more secure version of the SMTP port that implements asymmetric key encryption and TLS certificates to validate the email server is who they say they are. Most emails servers are configured to exchange emails over SMTPS where possible, but when not supported by both sides, emails can be exchanged over plaintext SMTP instead; this leaves the possibility of inspecting email traffic when the email server fallback to using SMPT instead of SMTPS.
  
- TCP 993 : IMAPS protocol, which is the same as the IMAP protocol, but supports TLS/SSL encryption between the client and the server, ensuring the email and credential information cannot be easily read if captured by a malicious threat actor.
  
- TCP 995 : POP3s protocol, which is the same as the pop3 protocol, but supports TLS/SSL encryption between the client and the server, ensuring the email and credential information cannot be easily read if captured by a malicious threat actor.

## Interpretation of Results:
1. The server is definitely an email server running the dovecot and postfix software – If we can find the specific version of these software that the target server is running, then we can search vulnerability websites/databases for potential flaws and security holes in the code which can be exploited.
2. The email server is also using insecure protocols (SMTP, IMAP and POP3), which we could potentially exploit via a MiTM attack in order to read the contents of emails and potentially user credentials for the email server.
3. There is some kind of website running on the webserver, as NGINX is listening on both HTTPand HTTPS – we could scan the website to try and find any vulnerabilities in the code of the website or any hidden pages that shouldn’t be accessible.
4. The server is running Debian Linux – if we can find the specific version then maybe we could look for vulnerabilities in the open-source code, which anybody with the correct skills can audit.


## Next Goal
Since services are running on the target server, SSH, Dovecot, Postfix and NGINX, we want to find more information about the versions of software running on the machine and any potential vulnerabilities which may exist. Unfortunately, without access to server via shell login, the next step is to attempt to gain the credentials of a user on the server.


## Step 3 : Credential Attack using Hydra
For a successful login via SSH, two conditions must be met: 
1) obtain the username 
2) obtain the password for that user account.

The Hydra tool is ran in dictionary attack mode, which allows us to specify text files which contain a list of potential usernames and a list of potential passwords. For usernames, a "userlist.txt" and for passwords "rockyou.txt" files are used. 

<img width="497" height="269" alt="3" src="https://github.com/user-attachments/assets/e4044026-3430-4b09-9d97-350d43abfff6" />

## Results

In the screenshot below, you can see that hydra found out that the username is “callum” and the password is “helloworld”,  and at this point we can try to login to the machine via SSH:

<img width="704" height="115" alt="4" src="https://github.com/user-attachments/assets/ac61a6fb-8263-4dc1-b57d-8b688196720c" />

 Access is gained intto the server via SSH, shown in the screenshot below. You can even see some file that was left in the users /home/ directory under webserver.config

 
<img width="683" height="278" alt="5" src="https://github.com/user-attachments/assets/29146ba4-02c0-488f-91aa-c6d4048d9783" />

From the screenshot below, it appears to be the configuration of an NGINX webserver, with it’s distinct use of the server
{} blocks in the config file, and it contains information about the public, private and root TLS certificate. If this file ends up on the wrong hands, it can be used to redirect users to a malicious website.

<img width="514" height="350" alt="6" src="https://github.com/user-attachments/assets/8034ac55-1ddc-4139-b508-1586e8b17ddf" />



## Step 3: Reconnaissance attack using OpenVas

The OpenVAS vulnerability management tool comes with an easy to use web interface which allows users of the software to customize the access and the methods used by the tool when conducting vulnerability scans. When creating a new scan task, the OpenVAS software provides the ability for users to enter SSH access credentials, and root credentials, so we provided those to the tool so that the software would provide us with as much information about the system as possible:

<img width="583" height="315" alt="7" src="https://github.com/user-attachments/assets/b86b14b5-576d-4767-b97b-280005db2aa3" />


<img width="581" height="317" alt="8" src="https://github.com/user-attachments/assets/32f81971-82db-48d3-9928-f76ba09ba938" />


## Results:

As soon as the scan finished, a report was available on the OpenVAS web interface:
<img width="584" height="307" alt="9" src="https://github.com/user-attachments/assets/f8472ca8-9d1b-4fea-ae51-4419e6f82e76" />

The report revealed 4 vulnerabilities within the target system and 2 open CVE’s:
<img width="583" height="305" alt="10" src="https://github.com/user-attachments/assets/2eb97823-d4a0-4fc4-a623-a787f119ba1b" />

<img width="584" height="307" alt="11" src="https://github.com/user-attachments/assets/5202431c-10a6-42d3-858b-c2eec83a9e6a" />


## Interpretation of Results:

- The first CVE, “SSL/TLS: Report ‘Anonymous’ Cipher Suites” (CVE-2007-1858, CVE-20140351) is listed as severity 5.4 (medium).
The vulnerability in this case could allow clients to negotiate a TLS/SSL session without any authentication or verification that the target server is who they say they are.To take advantage of this vulnerability, a malicious actor could make their machine appear as this mail server using the same hostname and keys (which they have access to with root privileges), and then wait for a connection from an authorised SSH user to the mail server. The malicious actor can then intercept the key exchange between the mail server and the valid SSH users host machine, make themselves appear as the mail server to the host, and as the host to the mail server, and then sniff the traffic of the SSH connection using a tool such as Wireshark or tcpdump. The listed solution for this CVE is to disable acceptance of ‘Anonymous’ cipher suites within OpenSSH, which will be discussed in the “security recommendations” section.

- The second CVE, “ICMP Timestamp Reply Information Disclosure” (CVE-1999-0524) is listed as severity 2.1 (low). A summary of this vulnerability is essentially that the target server is responding to an ICMP requests which asks for the timestamp of the machine clock at that very moment, essentially allowing a malicious actor to verify the current clock settings of the target server. It is rated as low because this kind of information by itself is hard to exploit, however it is listed on Tenable’s website that this “may assist in an unauthenticated, remote attacker in defeating time-based authentication protocols.” The listed solution for this CVE is to block incoming ICMP timestamp requests and outgoing replies from the server.

  ## Step 5: Quick Attempt of performing an SQL injection

A quick SQL injection attack attempt was made on the mailserver login screen hosted by
NGINX, as we suspected that it could be using an SQL backend database to store credentials. However, the SQL injection did not work in this scenario because of field requirements, thus we suspect that the developers of the RoundCube webmail package have coded the software to sanitize fields correctly.

<img width="577" height="375" alt="12" src="https://github.com/user-attachments/assets/44feb19b-7a56-448e-8e2d-4a727b0399f0" />



### Security recommendations for the target server
Some security recommendations which would significantly increase the security of the target server are as follows:

1.   User credentials on the server should be much more secure, and rotated on a regular basis! Brute-forcing the ssh login credentials using the hydra tool was too easy.


2.   SSH should be configured to only accept login using Trusted Known Host Keys. Using password authentication enables the possibility of a brute-force attack.


3.   A firewall, such as iptables, should be installed and configured to block all traffic inbound for network ports that shouldn’t be publicly available. The standard way of configuring this is to have a default “DENY ALL” traffic rule set on the INBOUND chain, and then whitelist services and IP ranges that should be allowed. This approach allows only the administrator-approved traffic into the device.

◦ This could be used to resolve CVE-1999-0524 as identified by the OpenVAS system, by blocking ICMP traffic, or more specifically ICMP packet types 13 and 14.

4.   An Intrusion Detection System, such as Fail2Ban, should be installed and implemented to monitor service logs and block bogus access attempts. This will dynamically ban IP’s that try to abuse the server.

5.   Services should be using secure versions of protocols where possible! For example, postfix and dovecot should be configured to use TLS/SSL certificates to encrypt traffic between client-server, and so should the NGINX service.

6.   Optionally, the web server could be configured to restrict HTTP/HTTPS access to only trusted subnets, so that random users cannot see the web interface and attempt attacks such as SQL injection.

7.   To resolve CVE’s: CVE-2007-1858 and CVE-2014-0351, the OpenSSH service should be configured via the /etc/sshd_config configuration file, and inside of this weak SSL/TLS ciphers should be disabled by deleting identified ciphers within the OpenVAS report.
