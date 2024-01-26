# Wireguard basics

## Install serever

```bash
sudo apt install wireguard
```

Next you can configure your wireguard interface

### Configure Wireguard interface

Run following command 

```bash
sudo nano /etc/wireguard/wg0.conf
```

and insert following content (edit server private key, client public key, ip addresses and whatever you need

```ini
# /etc/wireguard/wg0.conf
[Interface]
PrivateKey = <server-private-key>
Address = 10.0.6.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <client public key>
AllowedIPs = <all aloved ip addresses or networks divided by coma>
```

> To generate private and public key you can use `wg pubkey < peer_A.key > peer_A.pub`
{.is-info}

Following rules are optional and they are needed if you want to forward communicatoin to your local network.

```ini
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```

> `eth0` interface may differ from your.
{.is-info}

### Start WG interface

Start up interface to test it

```bash
wg-quick up wg0
```

To check statu of interface run `sudo wg`. It its everything OK enable it so its can run automatically after restat 

```bash
sudo systemctl enable wg-quick@wg0
```

### Firewall rules

enable port in firewall

```bash
sudo ufw allow 51820/udp
sudo ufw enable
```

### Internet forwarding

Run `sudo nano/sysctl.conf` and uncommend following rules

```ini
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```

> For security reason I recomend you to use different port for your server.
{.is-warning}


> If you want to allow forwarding to the internet edit `/etc/defaul/ufw` and change `DEFAULT_FORWARD_POLICY` from `DENY` to `ACCEPT`
{.is-info}
