# 🔌 Dokku mDNS Publisher Plugin

This is a plugin for [Dokku](https://dokku.com/) that automatically publishes mDNS domains on application deploys. My own use-case is to have my Dokku instance deployed on a Raspberry Pi auto-publish new apps on my home network.

## Requirements

- This plugin requires the host to be running `avahi-daemon`.
- Dokku version `>= 0.33.7`, it has not been tested on others.
- It has been tested on Raspbian, but in theory, it should also work with Debian and Ubuntu.

## Installation

```bash
# On the dokku server
dokku plugin:install https://github.com/calyhre/dokku-mdns.git
```

## Usage

After installing the plugin, it will automatically publish mDNS domains (`*.local`) whenever an application is created or deployed.

If you have the an mDNS domain (ex: `dokku.local`) set globally, for example, this will publish and advertise any `$app.dokku.local` automatically.

## What it does

1. On install, this plugin creates a new multi-instance service that takes an app name as an argument.
2. On app creation or deploy, it enables a new instance of that service with the app name as a variable.
   - The service queries all listed domains linked to the app, and takes the first mDNS one.
   - It tries to guess the primary network interface (`eth0`, `wlan0`, ...), and gets its IP address.
   - It finally starts `avahi-publish` and advertises the domain towards the address found before.
3. On app deletion, it deregisters the service.

## Remarks / Limitations

- Right now, the only way I found to "guess" an IP address to advertise to, is by using `route` and getting the default routing interface. I imagine this will not work properly if you have many interfaces, or a complex network setup.
- In order to allow the `dokku` user to restart a system service, I added 3 management scripts to the list of sudoers. These scripts accept an argument (in theory the app name), but I'm wondering if it's not possible to inject arbitrary commands through it, and have a sudo account run it.
- This has only been tested on Raspbian.
