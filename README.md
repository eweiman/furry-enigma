# furry-enigma
Hulu video ad blocking on FireTV devices via DNSmasq, lighttpd, minidnla, ddwrt


This is based on work originally using pixlserv to block ads using 1x1px GIFs and later incorporated some ideas from pi-hole. I've written this to run directly on your router to eliminate a need for extra devices on the network, and eliminate extra network hops 
which add latency and complexity. However it should be easily ported over to any \*nix based system on your network.  This inlcudes video ad blocking for Hulu on FireTV devices, it *may* work on other devices however it is not guaranteed. The way that Hulu runs 
things makes it difficult to have a one size fits all solution.  The FireTV will nearly instantly skip all ads in the broadcast, except for the network splash screen at start of show (think NBC splash and the time that the show airs). The times where you are asked to pick your ad experience will not be skipped entirely due to the timer, simply pick any experience and you will proceed to skip the ad. 

To block the video ad you need to replace the video that the Hulu app is requesting with a video of your own which makes the app believe that it displayed the proper advertisment instead of waiting 30 seconds to proceed. 


This makes the following assumptions: 
- That you are running a class A network with a class C subnet mask (not using 192.168.x.x).
- You _can_ change the IPs associated but it will take some time for the average person. 
- You have the "/jffs" partition on dd-wrt available. 
- You have minidnla running via ddwrt (under Services - NAS tab) enabled with the /jffs partition in the shared path where the video of your choice is located (smaller the better).

**You will need to use VLC to determine the video path for your replacement advertisement video and update the lighttpd configuration file for Hulu.** This should only need to be done once unless you change the video / create new database.

That your router is at the following IP:

- 10.10.10.1 
- That you have a Broadcom based router with interfaces named 'br0'
- That you do not have the IP addresses 10.10.10.254 and 10.10.10.253 being used on your network.
- That you are running a version of dd-wrt that inludes minidlna, dnsmasq, lighttpd.

Populate the 'startup commands' section of ddwrt with the following (without this you wont be serving adblocking webpages):
```
sleep 2
ifconfig br0:1 10.10.10.254 netmask 255.255.255.0
sleep 2
ifconfig br0:2 10.10.10.253 netmask 255.255.255.0
sleep 1
/jffs/adblock/lighttpd -f /jffs/adblock/lighttpd.conf
/jffs/hulu/lighttpd -f /jffs/hulu/lighttpd.conf
```


The startup commands listed will add extra IP addresses to your router that will serve as web pages that the ad requests will be re-directed to. 
The majority of websites will see the page resolve and accept that the ad is being shown. 
The 10.10.10.254 address is where normal ads will be sent to resolve.
The 10.10.10.253 address is where Hulu will be redirected to show your replacement video. 
Hulu blocking is only part of the main dnsmasq.conf file. 

You will need to make a copy of the lighttpd binary for each adblock type, one for basic ads and one for hulu. 

cp /usr/sbin/lighttpd /jffs/hulu/
cp /usr/sbin/lighttpd /jffs/adblock/

The web interface of dd-wrt will not indicate that you have enabled lighttpd and you don't need to enable it there either as we need to use custom configuration that we can't do from there. 

