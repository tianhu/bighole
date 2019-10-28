![picture](wireguard.png)

# Setup your own VPN server with WireGuard on Ubuntu

## Why WireGuard?

1. It is very simple to setup a WireGuard VPN server. I have ever setup kinds of VPN servers, such like L2TP, IKEv2 and OpenVPN. The WireGuard is the most simple VPN to be setup.

2. See below graphs.

![picture](wireguard-vpn.png)
![picture](wireguard-vpn-speed.png)

## Setup WireGuard server

Run below commands on the server.

1. `sudo add-apt-repository ppa:wireguard/wireguard`
2. `sudo apt install wireguard`
3. `sudo su`
4. `cd /etc/wireguard`
5. `umask 077`
6. `wg genkey | tee private.key | wg pubkey > public.key`

> * It will generate 2 files private.key and public.key that will be used later on both the server and the clients.

7. `vi wg0.conf`

> * Copy the content of server.conf from this Git repository.
> * Replace the `PrivateKey` value with the content of private.key which generated by previous command.
> * Change `eth0` if your network interface has another name.

8. `wg-quick up wg0`
9. `systemctl enable wg-quick@wg0`
10. `sysctl -w net.ipv4.ip_forward=1`
11. `sysctl -p`

## Setup WireGuard client

### Windows
1. Download WireGuard from https://www.wireguard.com/install/
2. Launch WireGuard.
3. Click `Add Tunnel` button.
4. Choose `Add empty tunnel...`.
5. Enter a name.
6. Append private `Address` and `DNS` to the `[interface]` section.
7. Add the server's public key and **public IP** address with port to a `[peer]` section.

It should look like below.

```
[Interface]
PrivateKey = iKB/c9cZCSYwGMGgJ6r/OJOBXaLgapgfhsrRZD+aGEo=
Address = 10.0.9.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = JbnFipIs+E8B1F9AniNd7OnPjGBTbO3iBGUgONmA+yM=
AllowedIPs = 0.0.0.0/0
Endpoint = <server-public-ip>:6001
```

Notes:

***
* Do not change the `PrivateKey` under the `[interface]` section.
* The `Address` value must be different with the server and other clients. In this sample, we set the server address to 10.0.9.1 and the first client to 10.0.9.2
* You can use other `DNS` addresses, such like `8.8.8.8` under the `[interface]` section.
* Replace the `PublicKey` value under the `[peer]` section with the content of the server's public.key.
* Set the server's **public IP** address with port to the `Endpoint` under the `[peer]` section.
***

### macOS

Same as Windows.

### Linux

Same as the server configuration without ListenPort and iptables settings of NAT forward.

### Android

1. Install WireGuard from App store.
2. Launch WireGuard.
3. Click Add button.
4. Choose `Create from scratch`.
5. Enter a name.
6. Click  `GENERATE` button to generate client's Private and Public keys.
7. Set private `addresses` to `10.0.9.3/24`. **It must be different with the server and other clients**
8. Set `DNS servers` to `1.1.1.1` **Or `8.8.8.8` as you like**
9. Just leave `Listen port` and `MTU` empty.
10. Click `ADD PEER` button to add the server information.
11. Copy paste the content of the `public.key` from the server to the Peer `Public key`.
12. Set `Allowed IPs` to `0.0.0.0/0`.
13. Set `Endpoint` to your server's **public IP** address in format `ip:port`. **The server's port is set in server's configration. In the guide we used 6001.**
14. Just leave other settings empty.

### iOS

Same as Android

## Allow clients connect to the server

You can use wg/wg-quick commands or modify the configuration file `wg0.conf`.

### Method 1 - use commands

1. `sudo wg set wg0 peer <client-public-key> allowed-ips 10.0.9.2/32`
2. `sudo wg-quick save wg0`

Notes:

***
* Replace `<client-public-key>` with the content of the client's PublicKey.
* Replace the `10.0.9.2/32` with the client **private IP**, not public ip, and use `/32` for its subnet mask.
***

### Or use method 2 - modify the configuration file

1. `sudo wg-quick down wg0`
2. `sudo vi /etc/wireguard/wg0.conf` 

***
Append below section.
```
[Peer]
PublicKey = <client-public-key>
AllowedIPs = 10.0.9.2/32
```

Notes:

* Replace `<client-public-key>` with the content of the client's public key.
* Replace the `10.0.9.2/32` with the client **private IP**, not public ip, and use `/32` for its subnet mask.
***

3. `sudo wg-quick up wg0`

## Miscellaneous

Unfortunately WireGuard is UDP only, you cannot configure it run over TCP. If you want to setup a TCP tunnel, refer to my another guide from https://github.com/tianhu/bigwall.
