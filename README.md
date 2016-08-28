# VPN Access Point

> Use the mini router [AR150](http://www.gl-inet.com/ar150/) and a VPN eg. [proxy.sh](https://proxy.sh/) to provide an always-private internet access point.

## Flash OpenWrt

Download the [firmware](https://wiki.openwrt.org/toh/gl-inet/gl-ar150#installation).

Download the [md5 sum file](https://downloads.openwrt.org/snapshots/trunk/ar71xx/generic/md5sums).

Calculate the md5 checksum (`md5 openwrt-ar71xx-generic-gl-ar150-squashfs-sysupgrade.bin`) and check if it matches the one in given in the `md5sums` file.

Flash the image via the system update web GUI of the router.

## Setup Openwrt

```sh
ssh root@192.168.1.1
opkg update
# install/enable the web GUI
opkg install luci nano openvpn-openssl luci-app-openvpn
/etc/init.d/uhttpd enable
/etc/init.d/uhttpd start
```

Go to 192.168.1.1, login and set a password.

Add you SSH key!

Uncheck `Allow SSH password authentication`.

--

> **Troubleshooting**
> If you lock yourself out eg. with wrong firewall routes continually press the `Reset` button. The connection will be available only while keeping the button pressed. Now you can ssh into the device and reset the device with `firstboot`.

--

## Setup Firewall Rules

Create a new network interface: Set the name to `tun0`, and protocol to `Unmanaged`. Set `Custom Interface:` to `tun0 `.

Create a new firewall rule: Set

- `Input`: `reject`, 
- `Forward`: `reject`,
- `Output`: `accept`

with `Masquerading` and `MSS clamping` enabled.

Select `tun0 ` under `Covered networks`. Enable `Allow forward from source zones:` for `lan`.

Disable forwarding from `lan` to `wan` and generally set `forwarding` to `reject` for `lan`.


## OpenVPN

`/etc/config/openvpn`

```sh
# Configured for usage with an account from
# https://proxy.sh/
config openvpn 'proxy_sh'
        option nobind '1'
        option proto 'udp'
        option verb '3'
        option comp_lzo 'yes'
        option client '1'
        list remote 'de.proxy.sh'
        option auth_user_pass '/etc/openvpn/userpass.txt'
        option cipher 'AES-256-CBC'
        option auth 'SHA512'
        option enabled '1'
        option tls_client '1'
        option ca '/etc/openvpn/ca.crt'
        option dev 'tun0'
        option persist_tun '1'
        option route_delay '2'
```

--

`/etc/openvpn/userpass.txt`

```sh
USERNAME
PASSWORD
```

```sh
chmod 600 /etc/openvpn/userpass.txt
```

--

`/etc/openvpn/ca.crt`
> The [proxy.sh](https://proxy.sh/) certificate.

```
-----BEGIN CERTIFICATE-----
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
TSMSym4Np+AmT7OX
-----END CERTIFICATE-----
```

```sh
chmod 644 /etc/openvpn/ca.crt
```

## Reboot

:rocket: We're done!

