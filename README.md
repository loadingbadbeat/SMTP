# SMTP
Everything SMTP


<b>SMTP Basics -</b>
we have an SMTP session between sender.example and receiver.example

![image](https://github.com/loadingbadbeat/SMTP/assets/45952458/ac66f8b3-bea1-4a26-b3dc-73bd64c0ce35)

<b>1.Initially, the receiving SMTP server mx.receiver.example sends a greeting, indicating that it is ready to accept incoming SMTP commands. <br>
<b>2.The "client"/sender sender.example initiates the SMTP session by sending the EHLO (extended hello) SMTP command to the server, specifying its domain name. Note that all SMTP commands will be highlighted in red. <br>
<b>3.mx.receiver.example responds with a 250 status code, indicating that the requested command (EHLO) was successful. It also provides supported SMTP capabilities. <br>
<b>4.Then, sender.example specifies the sender e-mail address as well as the receiver e-mail address via "mail FROM" and "rcpt TO" in the mail envelope. mx.receiver.example confirms both of these SMTP commands with a 250 status code. <br>
<b>5.Now, sender.example transmits the "data" command to signal its intention to start sending the message data/content. <br>
<b>6.After receiving a "go ahead" response, sender.example sends messageheaders "Subject"/"From"/"To", a message body "lorem ipsum" and an end-of-data sequence"<CR><LF>.<CR><LF>" (Carriage-Return / Line-Feed, and a single dot in a line). Note that all message data will be highlighted in blue. <br>
<b>7.mx.receiver.example reads the end-of-data sequence and responds with a 250 status code, implying that the message data was accepted. <br>
<b>8.Lastly, sender.example closes the connection via the "quit" command. <br>

![image](https://github.com/loadingbadbeat/SMTP/assets/45952458/b283c718-3928-40e1-9f7a-174ec01bb76c)

SMTP session transferring a mail object, including mail envelope and mail content/data, with SMTP commands, line breaks and data separated by color


When sending an e-mail via outlook.com, we generally have two options: 

Outlook's web interface 
Using SMTP 

Since this blog post is about SMTP smuggling and doesn't have to do anything with web applications (like SMTP header injection), we're opting for number two - SMTP! How this works is described in figure 3. 
![image](https://github.com/loadingbadbeat/SMTP/assets/45952458/ac5b1f2c-2023-4c46-acb2-5a04d29492b4)

<b>SPF, DKIM and DMARC </b>
Before an inbound SMTP server accepts an e-mail, it checks the sender's authenticity via e-mail authentication mechanisms such as SPF, DKIM and DMARC. This is important, since otherwise attackers could just send e-mails from arbitrary domains. For example, sending an e-mail as admin(at)outlook.com from an attacker server would be entirely possible. The most prevalent e-mail authentication mechanism, SPF, works by permitting sender IP addresses in special SPF/TXT DNS records. The SPF record of outlook.com permits the following IP ranges for e-mail transfer: 

v=spf1 include:spf-a.outlook.com include:spf-b.outlook.com ip4:157.55.9.128/25 include:spf.protection.outlook.com include:spf-a.hotmail.com include:_spf-ssg-b.microsoft.com include:_spf-ssg-c.microsoft.com ~all 

![image](https://github.com/loadingbadbeat/SMTP/assets/45952458/8de0aee7-e180-4577-b3f7-f6acf32271b8)

Passing SPF checks with attacker.example domain, while sending as user@sender.example

The same goes for DKIM (DomainKeys Identified Mail) as well. DKIM allows signing the message data, including the From header. This signature can then be verified by the receiver with a public key that resides in the DNS. However, it is not enforced, which domain holds the public key

![image](https://github.com/loadingbadbeat/SMTP/assets/45952458/bb9bec8f-e7ea-4916-80dc-2951d4f12273)
<br>Passing SPF and DKIM checks, by sending from an attacker server with an attacker-controlled DKIM key


The same goes for DKIM (DomainKeys Identified Mail) as well. DKIM allows signing the message data, including the From header. This signature can then be verified by the receiver with a public key that resides in the DNS. However, it is not enforced, which domain holds the public key<br>
![image](https://github.com/loadingbadbeat/SMTP/assets/45952458/ee79cadd-4f0f-4d44-a803-9a47291bcf9b)<br>


In this case, the message from "user@sender.example" may be signed, but the verification key is located at "dkim._domainkey.attacker.example". This location is derived from concatenating the selector (s=) "dkim", the static value "_domainkey" and the domain (d=) "attacker.example". So, even though an e-mail may have a valid SPF record and a valid DKIM signature, there is no mechanism for telling if the e-mail comes from a malicious sender or not. 

Fortunately, there is DMARC, which stands for "Domain-based Message Authentication, Reporting and Conformance". DMARC introduces so-called "identifier alignment" for SPF and DKIM and allows senders to specify alignment policies for both methods. It verifies if the email's "From" domain aligns with SPF checks and/or DKIM signatures. Thus, the DMARC check fails if there is a mismatch between the MAIL FROM and the From domain where otherwise the SPF check would pass. An example for a DMARC policy, which is always located in a TXT record at _dmarc.[domain], can be seen here: 

<b>v=DMARC1; p=reject; sp=none; fo=0; adkim=r; aspf=r; pct=100; rf=afrf</b>
