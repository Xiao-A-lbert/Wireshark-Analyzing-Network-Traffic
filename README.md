# Wireshark Analyzing Network Traffic

<h2>Description</h2>
In this task, I used wireshark to examine a sample pcap where an endpoint user was infected by a qakbot malware and found IOCs. 


<h2>Languages and Utilities Used</h2>

- <b>wireshark</b>
- <b>linux CLI</b>


<h2>Environments Used </h2>

- <b>Unbuntu</b> 

<br />
<br />
First changed time format to standard UTC.

![1) set to UTC standard](https://github.com/user-attachments/assets/36bb863d-f617-4a41-8616-aad0eb723522)

<br />
<br />
Next I looked at Capture File Properties to exammine hashes, length, time of capture, and total packets. 

![2) capture file properties for an overview of the pcap](https://github.com/user-attachments/assets/8e00f1c2-96db-4e1c-a8ab-56b350b8665b)

<br />
<br />  
I looked at conversation statistics to look at top ips of communication sorted by packets. Presumably the infected host os ip 10.0.0.149 and is reaching out to malicious 10.0.0.6. Other top 5 ips are also enumerated. 

![3) conversation stats to show victim ip and dst ip](https://github.com/user-attachments/assets/f0ad815c-7633-4a78-a744-2283f2b68c8d)

<br />
<br />
Looking at Protocol Hierarch Statistics, I can see 4 http apckets, along with 37 smtp packets. 

![4) small http traffic easy to dive in](https://github.com/user-attachments/assets/6240a71d-cd0a-415b-9a58-d03e477e5d39)

<br />
<br />
Before diving further, I added src and dst port columns. 

![5) adding src and dst port columns](https://github.com/user-attachments/assets/19b871be-8811-4ccd-a3e2-42d5a5797018)

<br />
<br />
Filtering for http traffic and following the get request for a .dat file. 

![6) filter http shows a get request for dat file follow http stream](https://github.com/user-attachments/assets/d5bcf628-d1ee-42db-ac3a-d8c55461154c)

<br />
<br />
This shows the user-agent using a curl command to get a 86607 .dat file. 

![7) following the stream shows the endpoint used curl to download file from sus ip](https://github.com/user-attachments/assets/2d1ce6fc-037d-49e8-8e2a-77f71cb7b20e)

<br />
<br />  
Similarly, going to statistics>http>requests will show all the http requests which enumerates the 86607 .dat file. 

![8) similaryly under statistics http request will how the dat file](https://github.com/user-attachments/assets/a4a86f64-346f-4902-83e5-564e36a13169)

<br />
<br />
I saved the file into a directory to show details of 86607 .dat being a PE32 executable and generating a sha256 hash for it. 

![9) saved object hash](https://github.com/user-attachments/assets/7c22bbd6-3fd5-481d-9852-2905e381882b)

<br />
<br />
Virustotal shows the file hash to be malicious with 57 vendor flags for qakbot. 

![10) virustotal says trojan](https://github.com/user-attachments/assets/e6e686f5-b12c-48e9-ba2b-bd34c15f81b2)

<br />
<br />
Looking up the hash on malware bazaar also shows a quakbot signature. 

![11) malware bazaar quakbot ioc](https://github.com/user-attachments/assets/614e8cf0-24fd-4bd7-b2c7-a7007e2bfa6a)

<br />
<br />
Doing some OSINT, cyber.gc.ca & MITRE shows Qakbot observed to perform ARP scans to discover endpoints on the network. 

![12) some osint shows quakbot performs arp scan discovery](https://github.com/user-attachments/assets/73cc65a3-d109-481f-9464-4a5d62fa069d)

<br />
<br />  
Filtering for arp and ethernet destination of a broadcast shows 10.0.0.149 performing a descending arp scan. 

![13) filtering for arp and eth dst of broadcast address shows descending arp scan](https://github.com/user-attachments/assets/ce87e8b1-1a1d-4ab7-b982-241eac799e3b)

<br />
<br />
Filtering for icmp, our endpoint victim 10.0.0.149 shows to ping 10.0.0.6 and 10.0.0.1.  

![14) attacker pinged two ips within the network](https://github.com/user-attachments/assets/d57bcbe8-808d-4511-898f-a50a4d08e46e)

<br />
<br />
Filtering specifically for ip 10.0.0.1 shows failed tcp connections over port 445, 139, and 80.

![15) failed tcp handshakes over port 445 139 80](https://github.com/user-attachments/assets/bb8fc9fb-4386-433c-810f-c6cb2e28f1af)

<br />
<br />
Moving on to smtp traffic, we see 37 packets one of which is a AUTH LOGIN.

![16) auth login](https://github.com/user-attachments/assets/9949e6fc-d83a-4055-a09c-208d1e205173)

<br />
<br />
Following the tcp stream of the AUTH LOGIN packet shows a suername and password when decoded from base 64. Even though the authentication failed, passwords may need to be reset for the user for compromised credentials. 

![17) possible compromised user credentials](https://github.com/user-attachments/assets/9a2f870f-409c-415a-8174-cbad03c5a957)

<br />
<br />
Looking at SMB objects shows 6 dll files associated from our malicious hostname ip 10.0.0.6 

![18) suspicious smb objects ](https://github.com/user-attachments/assets/ed90cf9a-57e7-4b78-9675-471d04b10e7f)

<br />
<br />
Saving those files and generating sha256 hashes matches the has for the 86607 .dat file priviously found. Further investigation may involve SIEM correlation. This concludes enumerating IOCs with wireshark from a sample pcap.

![19) hashes are same as quakbot found in http flow](https://github.com/user-attachments/assets/48603c20-cd44-4bd5-a806-579832f2aff7)

<br />
<br />
