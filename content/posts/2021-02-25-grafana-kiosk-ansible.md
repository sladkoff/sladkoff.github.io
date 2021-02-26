---
title: Setting up a Grafana Kiosk on a Raspberry Pi with Ansible
date: "2021-02-26"
comments: true
buyMeACoffee: true
draft: false
toc: false
tags:
- grafana
- kiosk
- raspberry
- pi
- ansible
---

I was bored, I had a spare TV screen and a Raspberry Pi 3. 
I also love watching colorful charts on [Grafana](https://grafana.com/) :chart_with_upwards_trend:

As I'm currently learning about [Ansible](https://www.ansible.com/), 
I decided to write a playbook that sets up a Headless Raspberry Pi in Grafana Kiosk mode.

This quick guide shows you how to create a Kiosk of your own.

You'll need:

- A Raspberry Pi with an empty SD card
- Ansible on your local machine
- A display or TV to connect to your Raspberry
- A Grafana instance that is reachable by the Raspberry

## Installing the OS

I used the GUI [rpi-imager](https://github.com/raspberrypi/rpi-imager) to bootstrap my SD card with 
**Raspberry Pi OS Lite (32bit)** for my Raspberry. I think it's quick and easy and offers you some distro options 
to choose from. 

You can get the tool from the AUR if you're using Arch:

```bash
yay -S rpi-imager
```

Here's a screenshot of the GUI (select an image, select SD card, write!):

![rpi-imager](/img/posts/grafana-kiosk-ansible/rpi-imager-screenshot.png)

To make the raspberry auto-connect to your WiFi on boot, create a new file `wpa_supplicant.conf` inside the `boot` 
directory of your flashed SD card with the following contents.

```text
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=<Insert 2 letter ISO 3166-1 country code here>

network={
 ssid="<Name of your wireless LAN>"
 psk="<Password for your wireless LAN>"
}
```

To enable remote access over ssh after boot, create an empty file called `ssh` inside the `boot` directory as well.

## Installing grafana-kiosk

[grafana-kiosk](https://github.com/grafana/grafana-kiosk) is a simple wrapper script that starts a fullscreen Chrome session 
and opens a configured Grafana URL with optional authentication. 
This Grafana URL usually points to a [Grafana Playlist](https://grafana.com/docs/grafana/latest/dashboards/playlist/) 
which switches between different Grafana dashboards.

I wrote a simple [Ansible role](https://github.com/sladkoff/ansible-role-grafana-kiosk) to set up Headless Raspberry Pi Grafana Kiosks :tm:.

### Ansible Playbook

Here's my personal [Ansible playbook](https://github.com/sladkoff/ansible-playbook-grafana-kiosk). 

It assumes that you have a fresh Raspberry with no graphical environment. Like what we've set up during "[Installing the OS]({{< ref "#installing-the-os" >}})".

The playbook will install and configure the following things to autostart on boot:

- openbox
- chromium
- grafana-kiosk

To get started on your own RPI, you can fork [my repo](https://github.com/sladkoff/ansible-playbook-grafana-kiosk) and 
apply at least the following changes:

#### roles/ssh-keys/tasks/main.yml

This role contains tasks to disable password login via ssh and upload ssh keys to the RPI. You can either get rid of
this if you don't need it or provide your own ssh keys:

```yaml
# roles/ssh-keys/tasks/main.yml
---
- name: Deploy SSH Key
  ansible.posix.authorized_key:
    user: pi
    state: present
    key: <URL-OR-FILE-TO-YOUR-SSH-KEYS>
    validate_certs: False
  notify:
    - Restart sshd

- name: Disable Password Authentication
  lineinfile:
    dest: /etc/ssh/sshd_config
    regexp: '^PasswordAuthentication'
    line: "PasswordAuthentication no"
    state: present
    backup: yes
  notify:
    - Restart sshd
```

#### hosts.yaml

Provide the IP of your RPI in the `hosts.yaml`:

```yaml
# hosts.yaml
grafana_kiosk:
  hosts:
    grafana_kiosk_rpi_tv:
      ansible_host: "<YOUR-RASPBERRY-IP-HERE>"

all:
  children:
    grafana_kiosk:
```

#### files/my-grafana-kiosk-config.yaml

Provide a configuration for the grafana-kiosk utility script, this is where you specify the path to the Grafana instance
and Playlist ID that you want to display on your kiosk:

```yaml
# my-grafana-kiosk-config.yaml
general:
  kiosk-mode: full
  autofit: true
  lxde: false

target:
  login-method: local
  username: pi-grafana-kiosk
  password: pi-grafana-kiosk-password
  playlist: true
  URL: https://<YOUR-GRAFANA-HOST-HERE>/playlists/play/<YOUR-GRAFANA-PLAYLIST-ID>?kiosk&autofitpanels
  ignore-certificate-errors: false

```

:play_or_pause_button: Run the playbook in the repo directory

```bash
ansible-galaxy install -r requirements.yaml
ansible-playbook install-grafana-kiosk -i hosts.yaml --ask-pass
```

You'll need to connecto the RPI to a screen and reboot it. If all went well, you should be greeted with your Grafana Playlist
after booting. 

If there are still issues with my Ansible voodoo, feel free to create a PR on Github :smile_cat: 