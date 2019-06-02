```
                     _   _
 _ __  ___ ___      | |_| |___
| '_ \/ __/ __|_____| __| / __|
| | | \__ \__ \_____| |_| \__ \
|_| |_|___/___/      \__|_|___/
```

![Build Status](https://codebuild.us-east-1.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiUzM4dGlsK2dPMmdoRkNQcjRjSU1JVmJENHNCTFFHVzVXSUQ0eWw2ajhYZVU3d0hhb2s0d0pzdzNNZUxSenc2Y1J3VmNyak9Udy91cUVsazlOR1h4WWJZPSIsIml2UGFyYW1ldGVyU3BlYyI6IjdFTWxobnVDRVVLbWNkUEYiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)

## Motivation

Unlike most web browser traffic, which is encrypted thanks to HTTPS, the resolving of domain names to internet addresses still happens through DNS, an old, unencrypted protocol. This benefits analytics companies, advertisers, internet providers and attackers, but not the end-user, who seeks online privacy and security.

## Overview

nss-tls is an alternative name resolving library for [Linux](http://www.kernel.org/) distributions with [glibc](https://www.gnu.org/software/libc/).

The glibc name resolver can be configured through nsswitch.conf(5) to use nss-tls instead of the DNS resolver, or fall back to DNS when nss-tls fails.

This way, all applications that use the standard resolver API (getaddrinfo(), gethostbyname(), etc'), are transparently migrated from DNS to encrypted means of name resolving, with zero application-side changes and minimal resource consumption footprint. However, nss-tls does not deal with applications that use their own, built-in DNS resolver.

## Architecture

nss-tls consists of three parts:

* nss-tlsd runs in the background and receives DNS resolving requests over a Unix socket.
* libnss_tls.so is a tiny client library which delegates the resolving work to nss-tlsd through the Unix socket and passes the results back to the application. This way, applications that take advantage of nss-tls are not affected by the complexity and the resource consumption of the libraries it depends on, or the constraints they impose on applications that use them.
* tlslookup is equivalent to nslookup(1), but uses libnss_tls.so instead of DNS.

nss-tls is designed with security and privacy in mind. Therefore, an unprivileged user can start a private, unprivileged instance of nss-tlsd and libnss-tls.so will automatically use that one, instead of the system-wide instance of nss-tlsd. Users who don't have such a private instance will continue to use the system-wide instance, which drops its privileges to greatly reduce its attack surface.

## Dependencies

nss-tls depends on:
* [glibc](https://www.gnu.org/software/libc/)
* [GLib](https://wiki.gnome.org/Projects/GLib)
* [libsoup](https://wiki.gnome.org/Projects/libsoup)
* [JSON-GLib](https://wiki.gnome.org/Projects/JsonGlib)

If [systemd](https://www.freedesktop.org/wiki/Software/systemd/) is present, the installation of nss-tls includes unit files for nss-tlsd.

However, nss-tlsd does not depend on [systemd](https://www.freedesktop.org/wiki/Software/systemd/). When [systemd](https://www.freedesktop.org/wiki/Software/systemd/) is not present, other means of running a nss-tlsd instance for each user (e.g. xinitrc) and root (e.g. an init script) should be used.

nss-tls uses [Meson](http://mesonbuild.com/) as its build system.

On [Debian](http://www.debian.org/) and derivatives, these dependencies can be obtained using:

1. apt install libglib2.0-dev libsoup2.4-dev libjson-glib-dev ninja-build python3-pip
2. pip3 install meson

## Usage

Assuming your system runs [systemd](https://www.freedesktop.org/wiki/Software/systemd/):

    meson --prefix=/usr --buildtype=release -Dstrip=true build
    ninja -C build install
    systemctl daemon-reload
    systemctl enable nss-tlsd
    systemctl start nss-tlsd
    systemctl --user --global enable nss-tlsd
    systemctl --user start nss-tlsd
    ldconfig

Then, add "tls" to the "hosts" entry in /etc/nsswitch.conf, before "dns" or anything else that contains "dns" in its name.

This will enable a system nss-tlsd instance for all non-interactive processes (which runs as an unprivileged user) and a private instance of nss-tlsd for each user.

## Choosing a DoH Server

By default, nss-tls performs encrypted name lookup over HTTPS through [1.1.1.1](https://developers.cloudflare.com/1.1.1.1/dns-over-https/).

To use a different DNS over HTTPS (DoH) server, use the "resolver" build option:

    meson configure -Dresolver=dns9.quad9.net/dns-query

## Performance

DNS over HTTPS is much slower than DNS. Therefore, it is recommended to enable cache of name lookup results. nscd(8) can do that.

To enable DNS cache on [Debian](http://www.debian.org/) and derivatives:

    apt install unscd

Set "enable-cache" for "hosts" to "yes" in /etc/nscd.conf. Then:

    systemctl enable unscd
    systemctl start unscd

## Legal Information

nss-tls is free and unencumbered software released under the terms of the GNU Lesser General Public License as published by the Free Software Foundation; either version 2.1 of the License, or (at your option) any later version license.

nss-tls is not affiliated with 1.1.1.1, [Cloudflare](https://www.cloudflare.com/) or [Quad9](https://www.quad9.net/).

The ASCII art logo at the top was made using [FIGlet](http://www.figlet.org/).