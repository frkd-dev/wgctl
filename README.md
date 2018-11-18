# WGCTL

Simple (70+ lines) bash script for basic WireGuard interfaces/peers management.

Install script:
```
wget https://raw.githubusercontent.com/stunpix/wgctl/master/wgctl > wgctl
chmod 755 wgctl && chown root:root wgctl && mv wgctl /usr/sbin/
```

**Note**: All actions require `root` priveleges.

Create vpn server interface configuration:

`wgctl addlink wg0 10.0.0.1/24`

Where `wg0` is name of vpn network interface in system, `10.0.0.1/24` vpn interface IP/netmask. Now add client/peer configuration:

`wgctl addpeer steve-iphone wg0 10.0.0.2/32`

Where `steve-iphone` is name of peer (choose any yours, but limit to alphanumeric), `10.0.0.2/32` peer's address and `wg0` is vpn interface to connect on.

You'll see textual confiuration for peer device and if you have installed `qrencode` you'll see config as QR code ready to use for mobile apps.

Bring vpn up: `wgctl up wg0`

Check status: `wg`

Stop vpn: `wgctl down wg0`

More options: `wgctl help`

### Make it permanent

Good practice is to manage your network interfaces with `ifupdown` toolset, especially when you need bring them up once system is (re)started. To do this create file `/etc/network/interfaces.d/wireguard` with following content:

```
auto wg0
iface wg0 inet manual
	up wgctl up $IFACE
	down wgctl down $IFACE
```

**Note**: make sure that `/etc/network/interfaces` has `source-directory /etc/network/interfaces.d` or similar line otherwise `ifupdown` will not see your interface. Some distors and VPS providers are wiping this line by overwriting `interfaces` file with they own version.

### Updating configs

To update interface or user configurations just run `wgctl addlink|addpeer` with same `NAME`/`IFACE` options. Old key/config files will be overwritten. Once you've done, reload vpn interface `ifdown wg0 && ifup wg0`.

**Note**: WireGuard doesn't have functionality to push new configs to your peer devices once configurations are updated, so you need to reconfigure them again. Run `wgctl showpeer <NAME> <IFACE>` to get fresh configs.

### Deleting peer configs

To delete all peer configs run `rm /etc/wireguard/<peer-name>.*`.

To delete peer for particular vpn interface run `rm /etc/wireguard/<peer-name>.<iface>.*`.

Once you've done – reload interface `ifdown wg0 && ifup wg0`.

### Deleting vpn interface

To delete vpn interface stop it with `ifdown wg0` then do `rm /etc/wireguard/<iface>.*`. Update `/etc/network/interfaces.d/wireguard` accordingly.

### Configuration files

Script holds all keys/configs in `/etc/wireguard` folder:

`*.key` private keys for local vpn interfaces.

`*.iface` local vpn interface configuration.

`*.peer` private keys and config files for vpn clients/peers.

### Compatibility

Tested on: Debian 9.

### License

MIT
