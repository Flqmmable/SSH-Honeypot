# SSH-Honeypot

This is a sort of storyboard of me creating my own SSH honeypot hooked up to a SIEM; it came with problems, but it was fun. :)

Originally, I wanted to host a honeypot on my own home network so I thought of different network topologies that would have the least effect on my current home devices; I came up with this.

![image](https://github.com/Flqmmable/SSH-Honeypot/assets/129753283/ad3e34a8-4fd7-4983-93f3-61b8a251edfd)

I would set up port forwarding (or so I thought) on my home router, and create rules of pfSense to allow traffic to reach the honeypot but block any traffic orginating from the honeypot from going to my internal home network - obviously to prevent a threat actor from moving laterally to my home devices.

It's also good to mention that I was using virtualisation so that I could host the firewall and the honeypot simultaneously on an old laptop and then connect them together via an internal-device network.

I downloaded openssh-server on the Ubuntu Server and set up all the rules on pfSense; I then connected the setup to my network as shown in the topology diagram.

## But then a problem! GCNAT :(

When setting up SSH port forwarding on my ISP's proprietary router, I noticed that I no outside SSH traffic was being redirected to pfSense in order for it to reach the honeypot. After doing some research, I found out about GCNAT (grade-carrier network address translation), and how most ISPs use it nowadays in order to cut the cost of providing publc IPs to their clients - it allows multiple clients to share the same public IP address. 

With GCNAT, an ISP groups together a set amount of their clients so that the WAN side of the client's home router is actually connected to an internal network which is shared by the rest of the clients in the same 'group'. At the 'far side' of this network, the ISP's router conducts NAT (PAT rather I assume) to give the client's in the group the same public IP address while still keeping track of which traffic needs to go to which client. With this in mind, it then becomes an impossibility that you can get SSH (or any type of) traffic to be sent to your home router because how is the ISP's router supposed to know which client send it to. Of course, if your ISP is kind enough, they could set up a port forwarding rule on their router to send particular traffic to your home router; but what if two or more clients want SSH to be sent to their router? See, it just wouldn't work out!

If you don't want to read what I wrote, in summary, GCNAT is port forwarding's worst nightmare 😂 and if you want to get rid of it, you'll have to pay your ISP to opt out of it and get a reserved public IP address - or 'business address' as they call it.

## New Solution - Cloud Servers

I would of liked the idea of having a honeypot set up on my own network, but oh well; I decided to go with cloud computing - more specifically, using AWS's EC2 for an Ubuntu Server.

EC2 was a good choice as it comes with a free tier option - with limited RAM and CPU power of course. The only issue I encounter was that, by default, you have to use key pairs to SSH (log into) your VPS; this was bad for me as I wanted to be able to see which usernames and passwords a threat actor tries to log in with. I overcame this problem by simplying enabling password authentication in the "sshd_config" file. 

![image](https://github.com/Flqmmable/SSH-Honeypot/assets/129753283/ef685ebc-d5ca-4d2c-9959-b41e996f2cde)

## Connecting to my SIEM

All that was left to do is hook it up with a SIEM; I chose Splunk to do this. I installed Splunk's Universal Forwarder onto the Ubuntu Server and then connected it to my cloud instance of Splunk. I added the "auth.log" file to the monitoring list ("auth.log" is the file which contains logs for SSH stuff) and boom! Any SSH activity is now sent to my Spunk instance. 

![image](https://github.com/Flqmmable/SSH-Honeypot/assets/129753283/fe4589e9-c274-48d3-9cc7-0e4aeff86d07)

As you can see, by running a simple query looking for any failed SSH logins, I can see, well.. failed SSH logins! I wanted to do more than this though; I wanted to create some sort of visualisation that shows where login attempts are coming from. 

But default, Splunk was not recongnising a "source IP" field so I had to use the "erex" command to allow Splunk to create a field for the source IP and pull out source IPs from the _raw field. 

![image](https://github.com/Flqmmable/SSH-Honeypot/assets/129753283/fa80a761-653d-4dc5-b006-34f82daf3639)

Eventually, I created a chloropleth map which visualises (on a map of the world) where SSH login attempts are originating from. Currently, the only SSH attempts are from me which is shown by the highlighted Australia.

![image](https://github.com/Flqmmable/SSH-Honeypot/assets/129753283/7926ec7e-ea96-4933-97f0-16e71f706c5a)

When you hover over a portion of the map, it shows you the count of attempts originating from there.

![image](https://github.com/Flqmmable/SSH-Honeypot/assets/129753283/23414ebd-5ee0-4b70-bea6-be503bbef30d)

## Conclusion

This text is just for documenting this cyber project of mine, but if someone is reading this, I hope you found it somewhat interesting :)

Since this is a honeypot, if I do get some attempts from potential attackers, I will be uploading screenshots of the Splunk visualisation for documentation purposes! 

Cya!





