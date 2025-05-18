For the first packet, I’ll look at the [exercise posted on 2025-01-22](https://www.malware-traffic-analysis.net/2025/01/22/index.html). The files can be downloaded from the website.

This exercise describes the following scenario:

You work as an analyst at a Security Operation Center (SOC). Someone contacts your team to report a coworker has downloaded a suspicious file after searching for Google Authenticator. The caller provides some information similar to social media posts at:

* [https://www.linkedin.com/posts/unit42\_2025-01-22-wednesday-a-malicious-ad-led-activity-7288213662329192450-ky3V/](https://www.linkedin.com/posts/unit42_2025-01-22-wednesday-a-malicious-ad-led-activity-7288213662329192450-ky3V/)  
* [https://x.com/Unit42\_Intel/status/1882448037030584611](https://x.com/Unit42_Intel/status/1882448037030584611)  
  Based on the caller's initial information, you confirm there was an infection.  You retrieve a packet capture (pcap) of the associated traffic.  Reviewing the traffic, you find several indicators matching details from a Github page referenced in the above social media posts.  After confirming an infection happened, you begin writing an incident report.

The exercise also gives information about the LAN segment from the pcap file, and finally provides 6 tasks. Let’s try and answer each task in turn\!

**What is the IP address of the infected Windows client?**

I think that the easiest way to find this is through Statistics \>\> Conversations. If I look at the IPv4 conversations, then almost every conversation lists the IP 10.1.17.215, as you can see marked in yellow in the below screenshot:

![image1](https://github.com/Stefan-Brewer/Home-lab-Wireshark-practice/blob/main/Pictures/20250122%2001.png)

A client is always included in it’s own conversations, and all the other IP addresses are the devices it is talking to.

**What is the mac address of the infected Windows client?**

We have the IP address, so we can right click on one of these conversations and select Apply as Filter \>\> Selected \>\> Filter on stream id, and then select a random packet.  
![image2](https://github.com/Stefan-Brewer/Home-lab-Wireshark-practice/blob/main/Pictures/20250122%2002.png)
And then look at the packet’s Ethernet layer data:  
![image3](https://github.com/Stefan-Brewer/Home-lab-Wireshark-practice/blob/main/Pictures/20250122%2003.png)
The source IP is 10.1.17.215, so that is the client. And if we look at the source MAC address, then we see that it is 00:d0:b7:26:4a:74.

**What is the host name of the infected Windows client?**

There are a number of ways to find the host name. First I’ll try to find it by searching for a DHCP packet. I can do that by simply entering dhcp into the search box of Wireshark. I found 4 packets, and if I look at the first one’s DHCP data packet, then there are a number of “Options”. Option (12) contains the host name, as you can see highlighted in yellow in the below screenshot:

![image4](https://github.com/Stefan-Brewer/Home-lab-Wireshark-practice/blob/main/Pictures/20250122%2004.png)

The host name is DESKTOP-L8C5GSJ

Other ways to find it is by looking for nbns (NetBIOS Name Service) packets:  
![image5](https://github.com/Stefan-Brewer/Home-lab-Wireshark-practice/blob/main/Pictures/20250122%2005.png)

SMB packets:  
![image6](https://github.com/Stefan-Brewer/Home-lab-Wireshark-practice/blob/main/Pictures/20250122%2006.png)

**What is the user account name from the infected Windows client?**

I found the account name by searching for a AS-REQ kerberos packet:  
![image7](https://github.com/Stefan-Brewer/Home-lab-Wireshark-practice/blob/main/Pictures/20250122%2007.png)
I was not able to find the username with any other methods.

**What is the likely domain name for the fake Google Authenticator page?**

Since this is about a domain name, we should look at DNS requests. Simply put dns in the search bar. This shows a lot of results related to “bluemoontuesday”, which is the domain in the scenario, and other domains from microsoft. To filter out domains that look trusted, I used the following seach:

dns and \!(dns.qry.name contains "bluemoontuesday" or dns.qry.name contains "microsoft" or dns.qry.name contains "msft" or dns.qry.name contains "bing" or dns.qry.name contains "msn")

This very quickly showed a number of suspicious domains from google-authenticator, which is what the client downloaded when they got infected. See below screenshot:  
![image8](https://github.com/Stefan-Brewer/Home-lab-Wireshark-practice/blob/main/Pictures/20250122%2008.png)
The domain has unrelated extra text (burleson-appliance.net) or is misspelled (authenticatoor.org).

The domain names that the fake Google authenticator app used was probably google-authenticator.burleson-appliance.net, which then may have redirected to the authenticatoor.org site.

**What are the IP addresses used for C2 servers for this infection?**

The above dns requests list a number of IP addresses. If I copy the text above from the DNS request, I get the following IP addresses from the google-authenticator.burleson-appliance domain:

104.21.64.1, 104.21.48.1, 104.21.32.1, 104.21.80.1, 104.21.16.1, 104.21.96.1, 104.21.112.1

Since the google-authenticator.burleson-appliance domain are all in the range 104.21.X.X, I did a search for ip.addr \== 104.21.0.0/16, and found that the only IP address that was actually used was 104.21.64.1. However, only 0.1% of all captures packets are related to this IP (42 packets).

The authenticator domain gives just one IP address: 82.221.136.26. There is a lot more traffic related to this IP: 6.3%, or 2470\.

It is not clear, but perhaps the initial google-authenticator.burleson-appliance downloaded the software that infected the client PC, and then that software started communicating with the authenticator domain? Anyway, the answer to the question is 104.21.64.1 and 82.221.136.26

**Checking the answers**

I have all answers correct except for the last one. So my guess is that the above IPs were just what downloaded the malware, and then the C2 servers are somewhere else.

I tried to do some more searching and could not find anything that might indicate what the C2 IP addresses might be, so I’ll just copy them from the answers file and search for the packets. Maybe I’ll learn something from how those packets behave.

**Why are these IP addresses suspicious?**

There are three C2 server IP addresses in the answers file. Let’s take a look at the first IP address, which is 5.252.153.241. These are 9077 packets sent between this IP and the client device. The protocol that is used a lot is HTTP. Maybe that is a hint? Not HTTPS, so not encrypted? The amount of data sent between this IP and the client IP is 7 MB, which is also a lot compared to most other conversations. Most of the data seems to be sent from the C2 server to the client, not the other way around. Additionally, the port numbers that are communicated with are in the very high digits, 49800+ range on the client IP side, and 2917 on the C2 server side. 

The next IP address is 45.125.66.32. There are 10,940 packets sent between this IP and the client device. This IP used HTTPS and the total amount of data is 10 MB, which is the most out of all conversations. The second most was the previous IP with 7 MB. Just like the previous IP, it is mostly data from the C2 server sent to the client. And again, port numbers that were used for the client side was in the 49,000+ range.

The next IP address is a lot less obvious. The conversations between 45.125.66.252 and the client only sent 107 kB using HTTPS. What is a bit of an outlier is the amount of packets that were needed for that 107 kB, which was more than a 1000 packets. And again, the port number was in the 49,800+ range. More specifically, all three IPs used ports between 49,800 and 50,000.

**Conclusion**

I feel like I learned a lot from this exercise. This is the first time I am using Wireshark to actually try and find something, and while I did miss the C2 server IPs, I found the answers to most of the exercises. Next time, it will be less intimidating, and I’ll be able to apply what I learned to hopefully find those C2 server IPs. Also, I asked Grok to roast my first report, and it gave me a lot of tips on what to look for, what I could improve upon, and how to structure SOC reports in the future.
