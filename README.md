# Community Hass.io Add-ons: WireGuard

[![GitHub Release][releases-shield]][releases]
![Project Stage][project-stage-shield]
[![License][license-shield]](LICENSE.md)

![Supports armhf Architecture][armhf-shield]
![Supports armv7 Architecture][armv7-shield]
![Supports aarch64 Architecture][aarch64-shield]
![Supports amd64 Architecture][amd64-shield]
![Supports i386 Architecture][i386-shield]

[![GitLab CI][gitlabci-shield]][gitlabci]
![Project Maintenance][maintenance-shield]
[![GitHub Activity][commits-shield]][commits]

[![Discord][discord-shield]][discord]
[![Community Forum][forum-shield]][forum]

[![Buy me a coffee][buymeacoffee-shield]][buymeacoffee]

[![Support my work on Patreon][patreon-shield]][patreon]

WireGuard: fast, modern, secure VPN tunnel.

## About

WireGuard® is an extremely simple yet fast and modern VPN that utilizes
state-of-the-art cryptography. It aims to be faster, simpler, leaner,
and more useful than IPsec, while avoiding the massive headache.

It intends to be considerably more performant than OpenVPN. WireGuard is
designed as a general purpose VPN for running on embedded interfaces and
super computers alike, fit for many different circumstances.

Initially released for the Linux kernel, it is now cross-platform (Windows,
macOS, BSD, iOS, Android) and widely deployable,
including via an Hass.io add-on!

WireGuard is currently under heavy development, but already it might be
regarded as the most secure, easiest to use, and simplest VPN solution
in the industry.

## Installation

The installation of this add-on is pretty straightforward and not different in
comparison to installing any other Hass.io add-on.

This is an installation manual & quick start:

1. [Add our Hass.io add-ons repository][repository] to your Hass.io instance.
1. Install the "WireGuard" add-on.
1. Set the `host` configuration option to your Hass.io (external) address,
    e.g., `myhome.duckdns.org`.
1. Change the name of the peer to something useful, e.g., `myphone`.
1. Save the configuration.
1. Start the "WireGuard" add-on
1. Check the logs of the "WireGuard" add-on to see if everything went well.
1. Forward port `51820` (UDP!) in your router to your Hass.io IP.
1. Download/Open the file `/ssl/wireguard/myphone/qrcode.png` stored on your
   Hass.io machine, e.g., using Samba, Visual Studio Code or the Configurator
   add-on.
1. Install the WireGuard app on your phone.
1. Add a new WireGuard connection to your phone, by scanning the QR code.
1. Connect!

**NOTE**: Do not add this repository to Hass.io, please use:
`https://github.com/hassio-addons/repository`.

## Configuration

**Note**: _Remember to restart the add-on when the configuration is changed._

If you are familiar with WireGuard, please note the following:
The configuration of WireGuard looks very similar to all terms used in the
WireGuard configuration. There is, however, one big difference: The
add-on is able to generate configurations for the add-on, but also for the
peers (clients).

Furthermore, the add-on takes care of a lot of default settings handling
for you. By default, only some basic settings are shown in the configuration
section of the add-on. However, more WireGuard setting are available!

An more extensive example add-on configuration, please do not copy paste.

```json
{
  "log_level": "info",
  "server": {
    "host": "hassio.local",
    "addresses": [
      "10.10.10.1"
    ],
    "dns": [
      "1.1.1.1",
      "1.0.0.1"
    ]
  },
  "peers": [
      {
        "name": "frenck",
        "addresses": [
          "10.10.10.2"
        ],
        "allowed_ips": [],
        "client_allowed_ips": []
      }
      {
        "name": "ninja",
        "addresses": [
          "10.10.10.2"
        ],
        "allowed_ips": [],
        "client_allowed_ips": [
          "10.10.10.0/24",
          "192.168.1.0/24"
        ]
      }
  ]
}
```

**Note**: _This is just an example, don't copy and paste it! Create your own!_

### Option: `server.host`

This configuration option is the hostname that your clients will use to connect
to your WireGuard add-on. The `host` is mainly used to generate client
configurations and SHOULD NOT contain a port. If you want to change the port,
use the "Network" section of the add-on configuration.

Example: `myhome.duckdns.org`, for local testing `hassio.local` will actually
work.

### Option: `server.addresses`

A list of IP (ipv4 or ipv6) addresses (optionally with CIDR masks) to be
assigned to the server/add-on interface.

It is strongly advised to create/use a separate IP address space from your
home network, e.g., if your home network uses `192.168.1.x` then DON'T use
that for the add-on.

### Option: `server.dns` _(optional)_

A list of DNS servers used by the add-on and the configuration generated for
the clients. This configuration option is optional, if no DNS servers are
set, it will use the build-in DNS server from Hass.io.

If you are running the [AdGuard][adguard] or [Pi-hole][pihole] add-on, you can
add the internal IP address of your Hass.io system to the list. This will
cause your clients to use those. What this does, it effectively making your
client (e.g., your mobile phone) having ad-filtering, while not at home.

### Option: `server.private_key` _(optional)_

Allows you to provide your own base64 private key generated by `wg genkey`.
This option supports the use of `!secret`. If you don't supply one,
the add-on will generate one for you and store it in:
`/ssl/wireguard/private_key`.

### Option: `server.public_key` _(optional)_

Allows you to provide your own a base64 public key calculated by `wg pubkey`
from a private key. This option supports the use of `!secret`.

If you don't supply one, the add-on will calculate one based on the private
key that was supplied via the `server.private_key` or, in case no private key
was supplied, calculated it from the generated private key.

### Option: `server.post_up` _(optional)_

Allows you to run commands after WireGuard has been started. This is useful
for modifing things like routing. If not provided, the add-on will by default
route all traffic coming in from the VPN through your home network.

If you like to disable that, setting this option to `true` (yes, true), will
disable that behaviour.

By default it executes the following:

`iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`

### Option: `server.post_down` _(optional)_

Allows you to run commands after WireGuard has been stopped. This is useful
for modifing things like routing. If not provided, the add-on will by default
remove the default rules created by the `post_up` defaults.

If you like to disable that, setting this option to `true` (yes, true), will
disable that behaviour.

By default it executes the following:

`iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE`

### Option: `peers.name`

This is an identifier for you. It helps you to know what this peer is, e.g.,
`myphone`, `mylaptop`, `ninja`.

This name is also used for creating the directory in `/ssl/wireguard` where
the generated client configuration and QR codes are stored. Therefor, a name
has a maximum of 32 characters, can only contains A-Z (or a-z) and 0-9.
Names contain a hyphen (-) but must not start or end with one.

### Option: `peers.addresses`

A list of IP (ipv4 or ipv6) addresses (optionally with CIDR masks) to be
assigned to the peer.

This is used in the client configuration, but also for used by the add-on to
set the allowed IPs (unless override by the `peers.allowed_ips` option.)

### Option: `peers.private_key` _(optional)_

Allows you to provide your own base64 private key generated by `wg genkey` for
the peer. This option supports the use of `!secret`.

Technically, the add-on does not need this, however, since the add-on can
generate client configurations, it can be helpful.

If no private key and no public key is provided, the add-on will generate one
for you and store it in: `/ssl/wireguard/<peer.name>/`.

### Option: `peers.public_key` _(optional)_

Allows you to provide your own a base64 public key calculated by `wg pubkey`
from a private key. This option supports the use of `!secret`.

If you don't supply one, the add-on will calculate one based on the private
key that was supplied via the `peer.private_key` or, in case no private key
was supplied, calculated it from the generated private key for this peer.

### Option: `peers.allowed_ips` _(optional)_

**This configuration only valid for the add-on/server end and does not
affect client configurations.!**

A list of IPs (ipv4 or ipv6) addresses (optionally with CIDR masks) from which
incoming traffic for this peer is allowed and to which outgoing traffic for
this peer is directed.

If there are no IP addresses configured, the add-on will use the addresses
listed in `peers.addresses`.

The catch-all `0.0.0.0/0` may be specified for matching all IPv4 addresses,
and `::/0` may be specified for matching all IPv6 addresses.

### Option: `peers.client_allowed_ips` _(optional)_

**This configuration only valid for the peer end/client configuration and does
not affect the server/add-on!**

A list of IPs (ipv4 or ipv6) addresses (optionally with CIDR masks) from which
incoming traffic from the server is allowed and to which outgoing traffic for
this peer is directed.

The catch-all `0.0.0.0/0` may be specified for matching all IPv4 addresses,
and `::/0` may be specified for matching all IPv6 addresses.

If not configured, the add-on will use `0.0.0.0/0` in the generate client
configuration, routing all traffic on your client though the VPN tunnel.

### Option: `peers.persistent_keep_alive` _(optional)_

A seconds interval, between 1 and 65535 inclusive, of how often to send an
authenticated empty packet to the peer for the purpose of keeping a stateful
firewall or NAT mapping valid persistently.

For example, if the interface very rarely sends traffic, but it might at
anytime receive traffic from a peer, and it is behind NAT, the interface might
benefit from having a persistent keepalive interval of 25 seconds.

If set to 0 or "off", this option is disabled.

By default or when unspecified, this option is off.
Most users will not need this.

### Option: `peers.endpoint` _(optional)_

An endpoint IP or hostname, followed by a colon, and then a port number. This
is used by the add-on/server to connect to its peer.

This is completely optional as the endpoint will be updated automatically
to the most recent source IP address and port of correctly authenticated
packets from the peer/client.

### Option: `peers.pre_shared_key` _(optional)_

A base64 preshared key generated by `wg genpsk`. This option adds an additional
layer of symmetric-key cryptography to be mixed into the already existing
public-key cryptography, for post-quantum resistance.

### Option: `log_level` _(optional)_

The `log_level` option controls the level of log output by the addon and can
be changed to be more or less verbose, which might be useful when you are
dealing with an unknown issue. Possible values are:

- `trace`: Show every detail, like all called internal functions.
- `debug`: Shows detailed debug information.
- `info`: Normal (usually) interesting events.
- `warning`: Exceptional occurrences that are not errors.
- `error`:  Runtime errors that do not require immediate action.
- `fatal`: Something went terribly wrong. Add-on becomes unusable.

Please note that each level automatically includes log messages from a
more severe level, e.g., `debug` also shows `info` messages. By default,
the `log_level` is set to `info`, which is the recommended setting unless
you are troubleshooting.

## Finding generated client configurations

All generated files are stored in `/ssl/wireguard`. This includes the
client configurations generated by this add-on.

Each peer/client will have its own folder, by the name specified in the
add-on configuration. The add-on additionally generates a image for each
client containing a QR code, to allow a quick an easy set up on, e.g., your
mobile phone.

## Running this add-on on a Generic/Debian/Ubuntu based Hass.io

The HassOS operating system for Hass.io by default has installed WireGuard
support in its Linux kernel. However, if you run Hass.io on a generic Linux
installation (e.g., based on Ubuntu or Debian), WireGuard support is not
available by default.

This will cause the add-on to throw a large warning during the start up.
However, the add-on will work as advertised!

When this happens, the add-on falls back on a standalone instance of WireGuard
running inside the add-on itself. This method has drawback in terms of
performance.

In order to run WireGuard optimal, you should install WireGuard on your
host system. The add-on will pick that up automatically on the next start.

### Ubuntu

```bash
sudo add-apt-repository ppa:wireguard/wireguard
sudo apt-get update
sudo apt-get install wireguard
```

### Debian

```bash
sudo echo "deb http://deb.debian.org/debian/ unstable main" > /etc/apt/sources.list.d/unstable.list
sudo printf 'Package: *\nPin: release a=unstable\nPin-Priority: 90\n' > /etc/apt/preferences.d/limit-unstable
sudo apt update
sudo apt install wireguard
```

### Fedora

```bash
sudo dnf copr enable jdoss/wireguard
sudo dnf install wireguard-dkms wireguard-tools
```

### Other

If you have a different operating system on which you run Hass.io on, please
consult [WireGuard installation manul][wireguard-install]

## _"Missing WireGuard kernel module. Falling back to slow userspace implementation."_

If you've seen this warning in the add-on logs files, please check the chapter
above for more information.

## _"IP forwarding is disabled on the host system!"_

IP forwarding is the ability for an operating system to accept incoming
network packets on one interface, recognize that it is not meant for the
system itself, but that it should be passed on to another network,
and then forwards it accordingly.

Basically, it allows for data coming in from your VPN client will be routed to
other places like your home network, or the internet.

## Snapshots

The WireGuard add-on can be backed up using Hass.io snapshots. There is one
caveat to take into account: A partial snapshot of the add-on DOES NOT contain
the generated client configurations, including their public/private keys.

Client configurations are stored in the `/ssl/wireguard` folder. If you
use partial snapshots, please besure to backup both the `ssl` folder and the
add-on.

## WireGuard status API

Lorem ipsum.

## Changelog & Releases

This repository keeps a change log using [GitHub's releases][releases]
functionality. The format of the log is based on
[Keep a Changelog][keepchangelog].

Releases are based on [Semantic Versioning][semver], and use the format
of ``MAJOR.MINOR.PATCH``. In a nutshell, the version will be incremented
based on the following:

- ``MAJOR``: Incompatible or major changes.
- ``MINOR``: Backwards-compatible new features and enhancements.
- ``PATCH``: Backwards-compatible bugfixes and package updates.

## Support

Got questions?

You have several options to get them answered:

- The [Community Hass.io Add-ons Discord chat server][discord] for add-on
  support and feature requests.
- The [Home Assistant Discord chat server][discord-ha] for general Home
  Assistant discussions and questions.
- The Home Assistant [Community Forum][forum].
- Join the [Reddit subreddit][reddit] in [/r/homeassistant][reddit]

You could also [open an issue here][issue] GitHub.

## Contributing

This is an active open-source project. We are always open to people who want to
use the code or contribute to it.

We have set up a separate document containing our
[contribution guidelines](CONTRIBUTING.md).

Thank you for being involved! :heart_eyes:

## Authors & contributors

The original setup of this repository is by [Franck Nijhof][frenck].

For a full list of all authors and contributors,
check [the contributor's page][contributors].

## We have got some Hass.io add-ons for you

Want some more functionality to your Hass.io Home Assistant instance?

We have created multiple add-ons for Hass.io. For a full list, check out
our [GitHub Repository][repository].

## License

MIT License

Copyright (c) 2019 Franck Nijhof

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

[aarch64-shield]: https://img.shields.io/badge/aarch64-yes-green.svg
[amd64-shield]: https://img.shields.io/badge/amd64-yes-green.svg
[armhf-shield]: https://img.shields.io/badge/armhf-yes-green.svg
[armv7-shield]: https://img.shields.io/badge/armv7-yes-green.svg
[buymeacoffee-shield]: https://www.buymeacoffee.com/assets/img/guidelines/download-assets-sm-2.svg
[buymeacoffee]: https://www.buymeacoffee.com/frenck
[commits-shield]: https://img.shields.io/github/commit-activity/y/hassio-addons/addon-wireguard.svg
[commits]: https://github.com/hassio-addons/addon-wireguard/commits/master
[contributors]: https://github.com/hassio-addons/addon-wireguard/graphs/contributors
[discord-ha]: https://discord.gg/c5DvZ4e
[discord-shield]: https://img.shields.io/discord/478094546522079232.svg
[discord]: https://discord.me/hassioaddons
[dockerhub]: https://hub.docker.com/r/hassioaddons/wireguard
[forum-shield]: https://img.shields.io/badge/community-forum-brightgreen.svg
[forum]: https://community.home-assistant.io/?u=frenck
[frenck]: https://github.com/frenck
[gitlabci-shield]: https://gitlab.com/hassio-addons/addon-wireguard/badges/master/pipeline.svg
[gitlabci]: https://gitlab.com/hassio-addons/addon-wireguard/pipelines
[home-assistant]: https://home-assistant.io
[i386-shield]: https://img.shields.io/badge/i386-yes-green.svg
[issue]: https://github.com/hassio-addons/addon-wireguard/issues
[keepchangelog]: http://keepachangelog.com/en/1.0.0/
[license-shield]: https://img.shields.io/github/license/hassio-addons/addon-wireguard.svg
[maintenance-shield]: https://img.shields.io/maintenance/yes/2019.svg
[patreon-shield]: https://www.frenck.nl/images/patreon.png
[patreon]: https://www.patreon.com/
[project-stage-shield]: https://img.shields.io/badge/project%20stage-experimental-yellow.svg
[reddit]: https://reddit.com/r/homeassistant
[releases-shield]: https://img.shields.io/github/release/hassio-addons/addon-wireguard.svg
[releases]: https://github.com/hassio-addons/addon-wireguard/releases
[repository]: https://github.com/hassio-addons/repository
[semver]: http://semver.org/spec/v2.0.0.htm
[adguard]: https://github.com/hassio-addons/addon-adguard-home
[pihole]: https://github.com/hassio-addons/addon-pi-hole
[wireguard-install]: https://www.wireguard.com/install/
