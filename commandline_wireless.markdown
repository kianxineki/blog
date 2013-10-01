# wireless on the command line

Connecting to wireless access points completely from the command line in linux
using the built-in tools is not actually very complicated. The hardest part
about it is turning off whatever "friendly" wireless/network managers your
system is already running.

# why the command line?

Graphical tools like `nm-applet` are handy but what they're doing is very
opaque. Sometimes you will tell them to connect to an access point but they will
ignore you and continue connecting to some other access point that you don't
want them to connect to. If you prefer to tell the computer exactly what to do,
managing wireless on the command line is actually not that hard or difficult and
you gain a lot of transparency into what your computer is doing to avoid
frustrating situations tinkering with opaque graphical tools.

Also if you like minimal or tiling windowing managers using a wireless applet by
way of something like stalonetray feels really awkward and strange.

# turning things off

## debian/ubuntu

```
$ sudo update-rc.d network-manager remove
$ pkill nm-applet
$ sudo service network-manager stop
```

or if `sudo service network-manager stop` didn't work, try:

```
$ sudo /etc/init.d/network-manager stop
```

If you're using a graphical environment with a panel that automatically spins up
something like nm-applet, you'll also need to figure out how to disable that
although it won't do anything if `network-manager` isn't running.

# figuring out the interface name

Type `iwconfig`. You will see a list of interfaces. Ignore all the interfaces
that say "no wireless extensions".

The interface name will be `wlan0`, `wlan2` or `ath0` or something like that.

This document uses the name `wlan0` but you should substitute `wlan0` for
whichever interface your system reports.

# adding passwords

```
$ sudo su
# wpa_passphrase SSID PASSPHRASE >> /etc/wpa_supplicant.conf
```

Make sure to use `>>` and not `>` or else you will delete all your wireless
passwords! It's a good idea to make a backup occasionally:

```
sudo cp /etc/wpa_supplicant.conf{,.backup}
```

# run wpa_supplicant

# scanning for access points

```
$ sudo iw dev wlan0 scan | grep SSID
    SSID: MEO-876078
    SSID: Thomson249040
    SSID: MEO-089464
    SSID: Solmar - Guests
    SSID: SINDICADO-NACIONAL
    SSID: Solmar
```

# connecting to an access point

To connect to an access point called SSID, do:

```
$ sudo iw dev wlan0 connect -w SSID
```

# see if you're connected to an access point

Use `iwconfig`:

```
$ iwconfig wlan0
```

When you're connected, you will see something like:

```
wlan0     IEEE 802.11abgn  ESSID:"Thomson249040"  
          Mode:Managed  Frequency:2.412 GHz  Access Point: 00:24:17:44:35:28   
          Bit Rate=48 Mb/s   Tx-Power=19 dBm   
          Retry limit:231   RTS thr:off   Fragment thr:off
          Power Management:off
          Link Quality=46/70  Signal level=-64 dBm  
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:170  Invalid misc:134   Missed beacon:0
```

# getting an IP address

Most of the time you'll just need to do:

```
sudo dhclient wlan0
```

but sometimes you will get the message:

```
RTNETLINK answers: File exists
```

In that case, release the dhcp lease first with `-r` and then get a lease:

```
$ sudo dhclient -r wlan0
$ sudo dhclient wlan0
```

Once dhclient finishes, you're online!

# disconnecting

```
sudo iw dev wlan0 disconnect
```

# see also

The [manual setup section](https://wiki.archlinux.org/index.php/Wireless_Setup_#Manual_setup)
of the archlinux wiki is very good but somewhat specific to arch in places.
