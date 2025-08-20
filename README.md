# wireguard-site-to-peer

## wireguard server and client setup on ubuntu 

## wireguard server credentials:
#### Azure vm 
- username: wireguardadmin
- password: Wireguard@123
- ipaddress: 98.70.102.68

##  Configuring a WireGuard server

### Installing WireGuard and Generating a Key Pair 
```
sudo apt update
sudo apt install wireguard
```
### creating and genearting the wireguard server’s key pair
```
wg genkey | sudo tee /etc/wireguard/private.key
```
create the public key, which is derived from the private key. Use the following command to create the public key file:
```
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```
### Choosing IPv4 Addresses Range Below To Assign For Wirgeguard Interface 
```
10.0.0.0 to 10.255.255.255 (10/8 prefix)
172.16.0.0 to 172.31.255.255 (172.16/12 prefix)
192.168.0.0 to 192.168.255.255 (192.168/16 prefix)
```
### Creating a WireGuard Server Configuration File

Once you have the required private key and IP address, create a new configuration file
```
sudo vim /etc/wireguard/wg0.conf 
```
This is config file of wireguard server. copy this file and paste in wg0.conf file
```
/etc/wireguard/wg0.conf
[Interface]
PrivateKey = base64_encoded_private_key_goes_here
Address = 172.16.0.1/24,
ListenPort = 51820
SaveConfig = true
```
Add the this below lines in server config file. these are used to enable and clean up NAT rules when the WireGuard interface is brought up or down, respectively 
```
PostUp = iptables -t nat -A POSTROUTING -s 172.16.0.0/24 -o eth0 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -s 172.16.0.0/24 -o eth0 -j MASQUERADE
```

after adding it should look like this 
```
[Interface]
Address = 172.16.0.3/24
SaveConfig = true
PostUp = iptables -t nat -A POSTROUTING -s 172.16.0.0/24 -o eth0 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -s 172.16.0.0/24 -o eth0 -j MASQUERADE
ListenPort = 51820
PrivateKey = cChD5ZWaPXN261R5Vr3mm44QDIp6t7ym17C98IeJZlo=
```

### Adjusting the WireGuard Server’s Network Configuration
To configure forwarding, open the /etc/sysctl.conf file 
```
sudo vim /etc/sysctl.conf
```
If you are using IPv4 with WireGuard, add the following line at the bottom of the file:
```
net.ipv4.ip_forward=1
```
To read the file and load the new values for your current terminal session, run
```
sudo sysctl -p
```
```
Output
net.ipv4.ip_forward = 1
```
Allow the ports on server and client machines according to requirement 
```
22 
51820 *
443
80
```
### Starting the WireGuard Server
```
sudo systemctl enable wg-quick@wg0.service
sudo systemctl start wg-quick@wg0.service
```


##  Configuring a WireGuard Peer
### Installing WireGuard and Generating a Key Pair 
```
sudo apt update
sudo apt install wireguard
```
### creating and genearting the wireguard peer’s key pair
```
wg genkey | sudo tee /etc/wireguard/private.key
```
create the public key, which is derived from the private key. Use the following command to create the public key file:
```
sudo cat /etc/wireguard/private.key | wg pubkey | sudo tee /etc/wireguard/public.key
```
### Creating the WireGuard Peer’s Configuration File
```
sudo nano /etc/wireguard/wg0.conf
```
This is the config file of wirreguard peer 
```
/etc/wireguard/wg0.conf
[Interface]
PrivateKey = base64_encoded_peer_private_key_goes_here
Address = 172.16.0.2/24

[Peer]
PublicKey = public_key_server
AllowedIPs = 172.16.0.0/24,0.0.0.0/0
Endpoint = 203.0.113.1:51820
```

### Adding the Peer’s Public Key to the WireGuard Server
Ensure that you have a copy of the base64 encoded public key for the WireGuard Peer by running
```
sudo cat /etc/wireguard/public.key
```
```
Output
PeURxj4Q75RaVhBKkRTpNsBPiPSGb5oQijgJsTa29hg=
```
Now log into the WireGuard server, and run the following command

```
sudo wg set wg0 peer PeURxj4Q75RaVhBKkRTpNsBPiPSGb5oQijgJsTa29hg= allowed-ips 172.16.0.2
```
Once you have run the command to add the peer, check the status of the tunnel on the server using the wg command:

```
sudo wg show
```
```
Output
interface: wg0
 public key: U9uE2kb/nrrzsEU58GD3pKFU3TLYDMCbetIsnV8eeFE=
 private key: (hidden)
 listening port: 51820

peer: PeURxj4Q75RaVhBKkRTpNsBPiPSGb5oQijgJsTa29hg=
 allowed ips: 171.16.0.2/32, fd0d:86fa:c3bc::/128
```

Check the connectivity by running command below:  
```
Ping 172.16.0.2 
``` 
And run the below command in client machine to check the internet is comming from server 
```
curl ifconfig.me 
```
it should the show the public ipaddress of server
```
98.42.86.2
```







