---
title: "Home Networking in Japan: Getting started with the OpenWRT One"
date: 2025-03-23
tags: ["openwrt", "ipv6", "map-e", "ddns", "wireguard"]
author: "Hans Christian Rudolph"
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
UseHugoToc: true
---


During the year-end season between Christmas and New Year, I usually spend a good amount of time on personal projects -- be it a piece of software, a creative endeavor, or some other tinkering I don't get around to during the year. This time, I set out to upgrade my home network and make the switch to *OpenWRT*.

Why OpenWRT? Because the way most Japanese ISPs, incl. NTT, deliver internet connectivity via fiber to the home is peculiar. Marketed as *IPoE* or *IPv6 Plus*, the underlying protocol is MAP-E ([RFC 7597](https://datatracker.ietf.org/doc/rfc7597/)), which tunnels IPv4 through IPv6[^1]. Since it does not exactly enjoy wide-spread use in other markets, you will be hard-pressed to find commercial home routers supporting it out-of-the-box. Even other popular open-source projects, such as *pfSense* or *OPNsense*, do not currently support it. Fortunately, OpenWRT does and the project introduced their first official hardware, the [OpenWRT One](https://openwrt.org/toh/openwrt/one), back in November 2024. Although it is certainly possible to install the software on third-party routers (the project Wiki has a [long list of supported devices](https://openwrt.org/toh/start)), I needed to buy a new device anyway and wanted to have a relatively hassle-free experience by going with the official one. Moreover, by buying one of these you are directly supporting the OpenWRT development team and thus, ensuring they can continue maintaining and further improving the project. What better way to dip your toes into open-source home networking?

Getting the device up and running could not have been easier. It comes pre-flashed, features a WiFi 6 access point, and the Luci web interface is easy enough to understand to configure basic internet connectivity -- if it were not for this pesky MAP-E protocol. A tutorial on GitHub by [fakemanhk](https://github.com/fakemanhk/openwrt-jp-ipoe) came in really handy here, explaining how to configure WAN and LAN interfaces, MAP-E rules, and finally test IPv6 connectivity. For my internet connection by OCN (NTT), this worked without problems, and once this configuration is in place you should have:

- An interface `WAN6` with a public IPv6 address
- An interface `WAN6MAPE` with a public IPv4 address (the one tunneled through the IPv6 connection)
- An interface `LAN` with a private IPv4 address (the one exposed to your home network).

With this setup, you should be able to connect your end devices to the `LAN` network with the OpenWRT router serving as gateway, DHCP server, and DNS resolver -- just like your usual home router. If required, you can create a wireless network using the built-in access point and attach it to the `LAN` network as well. For very modest networking needs, this is pretty much all you need.

-----

Next, I also wanted to make use of OpenWRT's ability to run a WireGuard endpoint on the router for secure remote access. There are many guides on this topic online -- I found the one from [Dariusz Więckiewicz](https://dariusz.wieckiewicz.org/en/installing-vpn-server-on-router-openwrt-wireguard) to be particularly useful -- but how to make it work with the MAP-E internet connection?

For this to work, you need to setup Dynamic DNS (DDNS) allowing you to resolve a static, memorable domain name to the ephemeral IP address of the OpenWRT router. I'm using [dynv6](https://dynv6.com), since it supports IPv6 and is free of charge, but the relevant [Wiki article](https://openwrt.org/docs/guide-user/services/ddns/client) lists a number of supported DDNS providers that you can pick and choose from. Depending on the chosen provider, you will need to install either the default OpenWRT `ddns-scripts` or a provider-specific alternative (e.g., for Cloudflare or NoIP).
Importantly, when using MAP-E, you will want to resolve the public IPv6 address of your router, **not the public IPv4 address**. Once you've configured the OpenWRT DDNS integration, you should see the running service alongside the reported IP address and the time of the last update in Luci under `Services > Dynamic DNS`.

From here, you should be able to follow the WireGuard setup guide linked above with the DDNS host name as your endpoint. This will create a new interface `WG0` as well as the required configuration for a peer to connect to it (e.g., your smartphone). WireGuard traffic will automatically use the `WAN6`/`WAN6MAPE` interface as a default gateway.

And that's it. Overall, getting started with OpenWRT was a lot less complicated than I initially thought. The MAP-E quirk requires you to jump through a few more hoops, but thankfully, others have faced the same challenge before and took the time to document their findings and solution.
This basic setup has served me well for the past three months and I have not encountered any significant issues. Even a software upgrade to a new major version went entirely smooth. In the meantime, I have made a few more changes, for example, I don't use the built-in WiFi, but a more capable stand-alone access point instead -- something to be covered in a future post.

[^1]: While you can opt to use via PPPoE for a pure IPv4 connection, the performance is not great, varies greatly during peak hours, and the fastest 10Gbps plans don't support it at all.