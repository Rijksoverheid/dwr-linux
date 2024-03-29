# Rijksoverheid for and by Linux enthusiasts

Linux support doesn't come out of the box, there are some hacks needed.

## Citrix Workspace App

**Note**: you can _only_ login with a softtoken (Digipass), see [the SSC-ICT manual](https://www.ssc-ict.nl/documenten/handleidingen/2020/06/08/handleiding---flexibel-werken-met-een-software-token). This activation can only be done from a location where you can access https://ssp-flex2rijk.frd.shsdir.nl/ . So you have to do this from a thin client (maybe managed hardware works too, but I don't have any gov managed hardware).

You can download a `.deb`, `.rpm` and `.tar` from  [Downloads / Citrix Workspace App](https://www.citrix.com/downloads/workspace-app/).

**Note:** _the current_ version 2104 from the 28th of April 2021 (`icaclient_21.4.0.11_amd64.deb`) somehow cripples the interprocess communication (IPC) of Firefox.
See [Ask Ubuntu](https://askubuntu.com/questions/1327810/20-04-firefox-not-rendering-or-loading-pages) and [Linux Mint Forums](https://forums.linuxmint.com/viewtopic.php?f=47&t=348798).
So stick with the older 2103 from the 9th of March 2021 (tested: no problems), or try the 2106 - Technology Preview from the 27th of May 2021 (untested).

Debian/Ubuntu procedure:
```bash
# manually download, or replace the gda: $ curl -OL 'https://downloads.citrix.com/19171/icaclient_21.3.0.38_amd64.deb?__gda__=***
sudo apt install "$PWD/icaclient_21.3.0.38_amd64.deb"
```

### Setup certificates

By default, Citrix Workspace App ships with a certificate store that doesn't trust https://www.flex2rijk.nl/, which is a problem.

There are multiple solutions.

1. Use your default certificate store:
```bash
sudo mv /opt/Citrix/ICAClient/keystore/cacerts{,-provided}
sudo ln -s /etc/ssl/certs /opt/Citrix/ICAClient/keystore/cacerts
```
2. Copy certificates to `/opt/Citrix/ICAClient/keystore/cacerts` and run `/opt/Citrix/ICAClient/util/ctx_rehash`
```bash
sudo cp -R /some/place/with/pem-files /opt/Citrix/ICAClient/keystore/cacerts
sudo /opt/Citrix/ICAClient/util/ctx_rehash
```
Source [Ask Ubuntu](https://askubuntu.com/a/302322) and [Citrix Support Knowledge Center](https://support.citrix.com/article/CTX231524).

### Setting HDPI

Since the beginning of 2021 Ivanti now forgets your Windows DPI / scaling settings after a lock, these locks are triggered if the session is idle for too long.
I found this quite annoying, SSC-ICT didn't see this as a problem and only wants to fix this as NSK (niet-standaard-klantvraag), which costs money.

This is my workaround.

First install xdotool:
```bash
sudo apt install xdotool
```
Then for scaling to 200% use:
```bash
xdotool search --onlyvisible --name "^DWR-Next-v1909 \(SSL/TLS Secured, 256 bit\)$" windowactivate %1 key --delay 1000 Tab key --delay 500 --clearmodifiers --repeat 3 "Super_L+d" key --delay 500 --clearmodifiers Menu key --delay 100 --repeat 2 Up key --delay 2000 Return key --delay 100 --repeat 2 Tab key --delay 100 Return key --delay 100 --repeat 4 Down key --delay 100 Return key --delay 100 "Alt+F4"; #DWRhdpi
```
To combine this with an auto unlock, install xsel.
```bash
sudo apt install xsel
```
Then this is my setup, my password comes from my KeePassXC password manager and is 32 chars long:
```bash
PW=$(xsel -b);if [ "${#PW}" != "32" ]; then >&2 echo "found length ${#PWD} on clipboard: $PWD"; else xdotool search --onlyvisible --name "^DWR-Next-v1909 \(SSL/TLS Secured, 256 bit\)$" windowactivate %1 key --delay 1000 --window %1 'Ctrl+Alt+Delete' type "$PW" && xdotool search --onlyvisible --name "^DWR-Next-v1909 \(SSL/TLS Secured, 256 bit\)$" key --delay 4000 --window %1 Return key --delay 1000 --window %1 Tab key --delay 500 --window %1 --clearmodifiers --repeat 3 "Super_L+d" key --delay 500 --window %1 --clearmodifiers Menu key --delay 100 --window %1 --repeat 2 Up key --delay 2000 --window %1 Return key --delay 100 --window %1 --repeat 2 Tab key --delay 100 --window %1 Return key --delay 100 --window %1 --repeat 4 Down key --delay 100 --window %1 Return key --delay 100 --window %1 "Alt+F4"; fi #DWRunlock-hdpi
```

#### Demo
https://user-images.githubusercontent.com/4252918/120599273-96ffca00-c447-11eb-8e75-02af28d8bc31.mp4

Please note: I could use 'Menu d' to go to Display settings, but sometimes this would shoot a 'd' to an open Outlook, deleting a message, so I replaced it with double Up.

### Filesystem Share Cleanup

The Citrix Workspace App will create `.access^` and `.attribute^` files in the drives and folders you share. This is a cleanup command:
```bash
find -name '.access^' -size 0 -type f -delete && find -name '.attribute^' -type f -delete
```

## WebEx

Finally since 2021-05-10 there is a native Linux app!  
There is a `.deb` and `.rpm`: [Download Webex](https://www.webex.com/downloads.html)

Debian/Ubuntu procedure:
```bash
curl -sSfLo '/tmp/webex.deb' 'https://binaries.webex.com/WebexDesktop-Ubuntu-Official-Package/Webex.deb'
sudo apt install '/tmp/webex.deb'
```

```bash
# Update WebEx oneliner: check installed deb package, extract deb package version from metadata with range requests, if there is an update, download and install
DEB_URL='https://binaries.webex.com/WebexDesktop-Ubuntu-Official-Package/Webex.deb';DEB_PKG='webex';dpkg --compare-versions $(dpkg-query -f '${Version}' -W "$DEB_PKG") lt $(curl -r "132-$(( $(curl -r "120-129" -sA '' "$DEB_URL") + 131 ))" -o - -sA '' "$DEB_URL" | tar -xzOf - './control' | grep -oP --color=never '^Version: \K.*$') && curl -sSfLo "/tmp/$DEB_PKG.deb" "$DEB_URL" && sudo apt install "/tmp/$DEB_PKG.deb"
```
Please note that virtual inputs don't work unless there is echo canceling on it.

Setup fake WebEx-proof mic:
```bash
pactl load-module module-null-sink sink_name=VirtualSink sink_properties=device.description=VirtualSink
pactl load-module module-null-sink sink_name=silence sink_properties=device.description="Silent_sink_for_echo_cancel"
pactl load-module module-echo-cancel sink_name=virtual-microphone source_name=virtual-microphone source_master=VirtualSink.monitor sink_master=silence aec_method=null source_properties=device.description=Virtual-Microphone sink_properties=device.description=Virtual-Microphone
```
Source [Unix Stack Exchange](https://unix.stackexchange.com/a/594698).

## Teams

Some other government organizations (e.g. <abbr title="Vereniging van Nederlandse Gemeenten">VNG</abbr>) or external suppliers use Microsoft Teams.

There is a `.deb` and `.rpm`: [Download Teams](https://www.microsoft.com/en-us/microsoft-teams/download-app)

Debian/Ubuntu procedure:
```bash
curl -sSfLo '/tmp/teams.deb' 'https://go.microsoft.com/fwlink/p/?LinkID=2112886' # version 1.4.00.13653
sudo apt install '/tmp/teams.deb'
```
Teams will setup `/etc/apt/sources.list.d/teams.list`:
```
### THIS FILE IS AUTOMATICALLY CONFIGURED ###
# You may comment out this entry, but any other modifications may be lost.
deb [arch=amd64] https://packages.microsoft.com/repos/ms-teams stable main
```
Plus write and trust `/etc/apt/trusted.gpg.d/microsoft.gpg` and setup `~/.config/autostart/teams.desktop`.

Note that the autostart is restored after opening of Teams, you have to disable 'Auto-start application' in Settings->General, probably you also want to disable 'On close, keep the application running'.

You can only trust the Microsoft key for their repo:
```bash
# move Microsoft (Release signing) from trusted for all to a keyring
sudo mv /etc/apt/trusted.gpg.d/microsoft.gpg /usr/share/keyrings/microsoft-keyring.gpg
# pin teams to signing key
sudo sed -i -r 's@\]@ signed-by=/usr/share/keyrings/microsoft-keyring.gpg]@;3s/([^#])/# Manually added signed-by pin\n\1/' /etc/apt/sources.list.d/teams.list
# update:
sudo apt update
# after an update the .list file is untouched, but the microsoft.gpg is added again
# from now on you can use: $ sudo apt upgrade && sudo rm /etc/apt/trusted.gpg.d/microsoft.gpg
# or just upgrade teams: $ sudo apt install teams && sudo rm /etc/apt/trusted.gpg.d/microsoft.gpg
```

Please note that the native Linux Teams app [doesn't support a resolution higher than 720p (1280×720)](https://microsoftteams.uservoice.com/forums/908686-bug-reports/suggestions/42405061-linux-application-shows-black-video-on-webcam-with), so an [Elgato Cam Link 4K won't work](https://docs.microsoft.com/en-us/answers/questions/404273/black-screen-in-teams-using-elgato-cam-link-4k-in.html). There are [certified devices](https://www.microsoft.com/en-us/microsoft-teams/across-devices/devices?rtc=1) for Microsoft Teams, although unsure if this is valid for the Linux client and all devices are supported in Linux. Using [Open Broadcaster Software (OBS)](https://obsproject.com/) version 27 supports creating a Virtual Webcam and has full screen capture options via PipeWire on Wayland.

```bash
# list all Video4Linux devices
v4l2-ctl --list-devices
# check what formats a device supports, see the above comments about 1280×720
v4l2-ctl -d /dev/video0 --list-formats-ext
```

As a backup you can use a browser, which does support 1080p (1920×1080), [but Firefox is not supported](https://support.microsoft.com/en-us/office/join-a-teams-meeting-on-an-unsupported-browser-daafdd3c-ac7a-4855-871b-9113bad15907) so you have to use a Chromium based browser.
