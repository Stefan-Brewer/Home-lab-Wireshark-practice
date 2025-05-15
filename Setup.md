Setup

Installed Oracle VirtualBox

Looking at malware, or analyzing packets containing malware on my private computer is a recipe for disaster, so let’s not do that. Instead, I’ll be using a virtual machine with Kali Linux installed. To use a virtual machine, I’ll first need a hypervisor. Luckily I already have Oracle VirtualBox installed. I got the Windows version from [their website’s Download page](https://www.virtualbox.org/wiki/Downloads). I installed it with default configurations.

Installing Kali Linux

To install Kali Linux, I went to the Kali website’s download page and chose [Virtual Machines](https://www.kali.org/get-kali/#kali-virtual-machines).There are a number of images available, all for different hypervisors. Since I am using VirtualBox, I chose that option. The download is a compressed folder that I had to extract. Once extracted, I simply ran VirtualBox, clicked on Add, and then selected the file I just extracted.  
The VM has Kali pre-installed, so I didn’t even have to go through any installation process (unlike with the other two VMs in the screenshot.) With iso files, clicking on New instead would be better, as it gives more setup and install options.

**Note**: I was initially not able to get any VMs to run because I had to enable SVM in the BIOS settings.

Testing Kali linux

I launched the VM, signed in with the default credentials listed on the Kali linux website (kali/kali), and look at that, wireshark has already been installed. Kali linux sure is convenient.  
I confirmed internet and DNS are working in the VM by running `ifconfig` and `ping google.com`. And then I ran `sudo update` and `upgrade` to make sure everything on the system is up to date (This took quite some time). With Kali linux installed and updated, Wireshark preloaded, and network verified, the VM is ready for packet analysis.

Getting some packets

Kali linux has Firefox installed, so I’ll visit [Malware-Traffic-Analysis.net](http://Malware-Traffic-Analysis.net) from inside the VM and download some packets. After all, downloading them to my host machine isn’t going to work, as the host machine does not have Wireshark installed, and I don’t want packets that contain malware to be on my personal computer. They are just packets, so I’m sure it’s not something that poses any danger, but you never know.

The Malware-Traffic-Analysis website has two types of posts:

1. Post that contain real life malware cases. It shows you what happened, and you can download files such as the captured packets, the email file, images, etc.

2. Posts that contain real life malware cases with assignments. You can download the package, and there will be assignments of what to look for, with a link to the answers.

The assignments sound like a great resource, as they were likely chosen for beginner difficulty, and they give me a goal of what to look for. Once I am a bit more familiar, I can look at the other packets that have less direction.  
The most recent assignment is from 2025-01-22. The assignment described someone downloading a suspicious file after searching for Google Authenticator. I have downloaded the zip that contains the pcap file, and put the pcap file in Documents/Malware Traffic Analysis (“Malware Traffic Analysis” is a folder I made to keep these assignments in.) Note that the files are password protected. You can get the password from the site’s [about page](https://www.malware-traffic-analysis.net/about.html).
