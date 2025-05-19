# **SOC Incident Report: Malware Infection Analysis**

**Context**  
This is my second ever pcap analysis. I am getting the pcaps from [https://malware-traffic-analysis.net](https://malware-traffic-analysis.net), specifically the exercise posted on [2024-11-26](https://malware-traffic-analysis.net/2024/11/26). I will try to implement the advice I received regarding my previous analysis.

## **Incident Overview**

In this scenario, we work as an analyst at a Security Operation Center (SOC) for a medical research facility specializing in nemotodes. On November 26th 2024, alerts on traffic in the network indicated someone has been infected. After investigating, we found a malicious download that started beaconing a C2 server every 60 seconds.

## **Initial Infection Vector**

The victim downloaded a file from modandcrackedapk.com. It is unclear whether this was due to a link in a phishing email or due to a malicious website.

## **Indicators of Compromise (IoCs)**

### **Affected System**

IP address: 10.11.26.183  
MAC address:d0:57:7b:ce:fc:8b  
Host name: DESKTOP-B8TQK49  
Username: oboomwald  
Subnet: 10.11.26.0/24, communicating with default gateway 10.11.26.3

### **Malicious Domains**

modandcrackedapk.com, which leads to IP 193.42.38.139, from where a file of 11 MB was downloaded. VirusTotal gives the domain a 12 score, many services say it is phishing or malware.

**C2 Servers**:

193.42.38.139 \- There are multiple downloads, mostly in three larger chunks, totalling 11MB. This was likely what cause the infection.  
194.180.191.64 \- This starts 30 seconds after the above 11 MB download, beaconing every 60 seconds. VirusTotal gives the IP a score of 7\.

## **Analysis**

### **Methodology**

The pcap was analysed using Wireshark. The 5 IP addresses that we were alerted about were all investigated. Most of them are short conversations with trusted sources, but two of them turned out to be malicious.  
Used Statistics \>\> Conversations and found one conversation that sent the client a large amount of data, likely a download. This IP matched one of the IPs that we were alerted about. We found the domain using a dns search, and confirmed it to be malicious using VirusTotal  
Used a ip.addr search and found that one of the IPs we were alerted about started a very consistent 60 second pattern, just 30 seconds after the download described above.  
The malware file download was encrypted, so we were unable to export it.

### **Key Findings** 

* The infection began due to the user clicking on a link: modandcrackedapk.com. It is unclear whether this link was located in a phishing link or a malicious website.  
* The URL linked to 193.42.38.139, which sent 11 MB of data to the machine.  
* 30 seconds later, the infected machine then started to beacon every 60 seconds to the IP 194.180.191.64.

## **Recommendations**

1. **Containment**:  
* Remove DESKTOP-B8TQK49 from the network so it cannot infect further machines.  
* Examine any other machines for beaconing to the IP 194.180.191.64 to confirm whether the infection has spread. If it did, remove those machines from the network as well.  
2. **Eradication**:  
* Try to get the file that DESKTOP-B8TQK49 downloaded so we can analyze it.  
* Reimage DESKTOP-B8TQK49 and any other infected systems.  
3. **Detection**:  
* If you were able to get the malware file and analyze it, add its signature to any IPS/IDS systems  
4. **Prevention**:  
* Filter the URL modandcrackedapk.com and the IPs 193.42.38.139 and 194.180.191.64.  
* Educate employees on detecting malware website and phishing emails.  
5. **Further Analysis**:  
* Monitor for 60 second beaconing behavior of other IPs

## **Conclusion**

The analyzed pcap confirms a malware infection. The infected client DESKTOP-B8TQK49 started beaconing every 60 seconds with 194.180.191.64 shortly after infection, posing a risk of data exfiltration and further compromise. We recommend immediate containment and eradication to prevent data from being stolen, and to improve detection and prevention methods to stop future incidents.

**AI feedback**  
I asked Grok again to give feedback, and he gave me the following pointers:  
**Trace the Infection Path**: Use HTTP referers, DNS queries, or redirect chains to pinpoint how the user reached modandcrackedapk.com.  
**Deepen C2 Analysis**: Inspect beacon protocols, ports, packet sizes, and timing. Use Wireshark’s IO Graph to visualize the 60-second pattern.  
**Work Around Encryption**: Even with encrypted downloads, extract metadata (file size, MIME type) or analyze TLS certificates for anomalies.  
**Enrich with Threat Intel**: Dig into VirusTotal’s behavior tab, cross-reference IPs with threat feeds, and check for malware family links.  
**Beef Up IoCs**: Include file hashes, URLs, user-agents, and certificate details. Document all alerted IPs, even if ruled out.  
**Sharpen Recommendations**: Specify tools, filters, or techniques (e.g., Suricata rules, memory dumps). Tailor prevention to the org’s context (e.g., medical research).  
**Leverage Context**: Hypothesize the malware’s goal based on the target (e.g., data theft for nematode research IP).  
