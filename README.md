# WGCTL

Simple (under 100 lines) bash script for basic WireGuard interfaces/peers management using generated config files with support of nearly all configuration options: keys, addresses, ports, dns, allowed ips, keepalive, preshared-keys and QR encoded configs for mobile peers.

Install script:
```
wget https://raw.githubusercontent.com/stunpix/wgctl/master/wgctl > wgctl
chmod 755 wgctl && chown root:root wgctl && mv wgctl /usr/bin/
```

**Note**: All actions require `root` priveleges.

Steps to bring vpn up. 

1. Create vpn interface configuration: `wgctl addlink wg0 10.0.0.1/24`. Where `wg0` is vpn interface name in your system and `10.0.0.1/24` — interface IP/netmask.

2. Add client/peer configuration: `wgctl addpeer steve-iphone 10.0.0.2/32 wg0`. Where `steve-iphone` is name of peer (choose yours), `10.0.0.2/32` peer's address and `wg0` is vpn interface to which it belongs.

3. Bring vpn up: `wgctl up wg0`

4. Check status: `wg`

5. Get peer's configuration: `wgctl showpeer steve-iphone`. QR available only when you have installed `qrencode` tool.

Stop vpn: `wgctl down wg0`

Delete peer: `wgctl rmpeer steve-iphone`. This deletes peer from any active connection without stopping it and deletes peer's configuration files.

More options: `wgctl help`

### Make VPN permanent

Good practice is to manage your network interfaces with `ifupdown` toolset, especially when you need bring them up once system is (re)started. To do this create file `/etc/network/interfaces.d/wireguard` with following content:

```
auto wg0
iface wg0 inet manual
	up wgctl up $IFACE
	down wgctl down $IFACE
```

**Note**: make sure that `/etc/network/interfaces` has `source-directory /etc/network/interfaces.d` or similar line otherwise `ifupdown` will not see your interface. Some distors and VPS providers are wiping this line by overwriting `interfaces` file with they own version.

**Important**: this is not suitable for busybox environments, for busybox use `/etc/network/interfaces` file to store vpn interface configuration.

### Updating configs

Before updating existing configs — stop vpn interface like `ifdown wg0`. To update interface/user configurations run `wgctl addlink|addpeer` with same `NAME`/`IFACE` options as before. Old keys/config files will be overwritten. Once you've done, bring vpn interface up with `ifup wg0`.

**Note**: WireGuard doesn't have functionality to push updated configs to peer devices, so you need update them manually. Run `wgctl showpeer <NAME>` to get updated configs.

### Deleting vpn interface

To delete vpn interface stop it with `ifdown wg0` then do `rm /etc/wireguard/wgctl/<iface>.*`. Update `/etc/network/interfaces.d/wireguard` accordingly.

### Configuration files

Script holds all keys/configs in `/etc/wireguard/wgctl` folder:

`*.key` private keys for local vpn interfaces.

`*.iface` local vpn interface configuration.

`*.peer` client/peer configuration with private keys.

`*.psk` peer's preshared key.

### TODO

 * Updating peers with `wgctl updpeer` without restarting interfaces.
 * Deleting interfaces with `wgctl rmlink`.

### License

MIT
