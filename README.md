# WGCTL

Simple (~100 lines) bash script for basic WireGuard interfaces/peers management using generated config files with nearly all configuration options support: keys, addresses, ports, dns, allowed ips, keepalive, preshared-keys and QR encoded configs for mobile peers.

### Install

Prerequirements:

1. Linux.
2. All actions are running with `root` priveleges.
2. You have [installed][1] wireguard tools and kernel modules.

[1]: https://www.wireguard.com/install/

Steps to bring vpn up. 

1. Get the script:
```
wget https://raw.githubusercontent.com/stunpix/wgctl/master/wgctl
chmod 755 wgctl && chown root:root wgctl && mv wgctl /usr/bin/
```

2. Create vpn interface configuration: `wgctl addlink wg0 10.0.0.1/24`. Where `wg0` is vpn interface name in your system and `10.0.0.1/24` is vpn interface IP/netmask. You could choose any vpn interface name as: vpn0, mynet1, wgrd0, etc. We'll use `wg0`.

3. Add client/peer configuration: `wgctl addpeer steve-iphone 10.0.0.2/32 wg0`. Where `steve-iphone` is name of peer (choose yours), `10.0.0.2/32` peer's address and `wg0` is vpn interface to which it belongs.

4. Bring vpn up: `wgctl up wg0`

5. Check status: `wg`

6. Get peer's configuration: `wgctl showpeer steve-iphone`. QR available only when you have installed `qrencode` tool.

Stop vpn: `wgctl down wg0`

Delete peer: `wgctl rmpeer steve-iphone`. This deletes peer from any active connection without stopping it and deletes peer's configuration files.

More options: `wgctl help`

### Make VPN permanent

For Debian based distros the most preferred way is to manage your network interfaces with `ifupdown` toolset. To make vpn interface up across reboots create file `/etc/network/interfaces.d/wireguard` with following content:

```
auto wg0
iface wg0 inet manual
	up wgctl up $IFACE
	down wgctl down $IFACE
```

Make sure that `/etc/network/interfaces` has `source-directory /etc/network/interfaces.d` or similar line otherwise `ifupdown` will not see your interface. Some distors and virtual server providers are wiping this line in their `interfaces` files.

Busybox have its own implementation of `ifupdown` toolset, which have no support for `source-directory` option, so for such environments store your vpn interface configuration directly in `/etc/network/interfaces` file.

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

 * Crossplatform support with wireguard-go
 * Updating peers with `wgctl updpeer` without restarting interfaces.
 * Deleting interfaces with `wgctl rmlink`.

### License

MIT
