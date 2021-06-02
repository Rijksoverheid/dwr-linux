# Rijksoverheid for and by Linux enthusiasts

Linux support doesn't come out of the box, there are some hacks needed.

## Citrix Workspace App

You can download a `.deb`, `.rpm` and `.tar` from  [Downloads / Citrix Workspace App](https://www.citrix.com/downloads/workspace-app/).

**Note:** _the current_ version 2104 from the 28th of April 2021 (`icaclient_21.4.0.11_amd64.deb`) somehow cripples the interprocess communication (IPC) of Firefox.  
See [Ask Ubuntu](https://askubuntu.com/questions/1327810/20-04-firefox-not-rendering-or-loading-pages) and [Linux Mint Forums](https://forums.linuxmint.com/viewtopic.php?f=47&t=348798).  
So stick with the older 2103 from the 9th of March 2021 (tested: no problems), or try the 2106 - Technology Preview from 27th of May 2021 (untested).

Debian/Ubuntu procedure:
```bash
# manually download, or replace the gda: $ curl -OL 'https://downloads.citrix.com/19171/icaclient_21.3.0.38_amd64.deb?__gda__=***
sudo apt install "$PWD/icaclient_21.3.0.38_amd64.deb"
```

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
PWD=$(xsel -b);if [ "${#PWD}" != "32" ]; then >&2 echo "found length ${#PWD} on clipboard: $PWD"; else xdotool search --onlyvisible --name "^DWR-Next-v1909 \(SSL/TLS Secured, 256 bit\)$" windowactivate %1 key --delay 1000 --window %1 'Ctrl+Alt+Delete' type "$PWD" && xdotool search --onlyvisible --name "^DWR-Next-v1909 \(SSL/TLS Secured, 256 bit\)$" key --delay 4000 --window %1 Return key --delay 1000 --window %1 Tab key --delay 500 --window %1 --clearmodifiers --repeat 3 "Super_L+d" key --delay 500 --window %1 --clearmodifiers Menu key --delay 100 --window %1 --repeat 2 Up key --delay 2000 --window %1 Return key --delay 100 --window %1 --repeat 2 Tab key --delay 100 --window %1 Return key --delay 100 --window %1 --repeat 4 Down key --delay 100 --window %1 Return key --delay 100 --window %1 "Alt+F4"; fi #DWRunlock-hdpi
```

Please note: I could use 'Menu d' to go to Display settings, but sometimes this would shoot a 'd' to an open Outlook, deleting a message, so I replaced it with double Up.

## WebEx

Finally since 2021-05-10 there is a native Linux app!  
There is a `.deb` and `.rpm`: [Download Webex](https://www.webex.com/downloads.html)

Debian/Ubuntu procedure:
```bash
curl -OL 'https://binaries.webex.com/WebexDesktop-Ubuntu-Official-Package/Webex.deb'
sudo apt install "$PWD/Webex.deb"
```

Please note that virtual inputs don't work unless there is echo canceling on it.

Setup fake WebEx-proof mic:
```bash
pactl load-module module-null-sink sink_name=VirtualSink sink_properties=device.description=VirtualSink && \
pactl load-module module-null-sink sink_name=silence sink_properties=device.description="Silent_sink_for_echo_cancel" && \
pactl load-module module-echo-cancel sink_name=virtual-microphone source_name=virtual-microphone source_master=VirtualSink.monitor sink_master=silence aec_method=null source_properties=device.description=Virtual-Microphone sink_properties=device.description=Virtual-Microphone
```
Source [Unix Stack Exchange](https://unix.stackexchange.com/a/594698).
