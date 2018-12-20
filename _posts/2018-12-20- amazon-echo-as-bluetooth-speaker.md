---
layout: post
title: Use Amazon Echo as Bluetooth Speaker / Audio sink in Ubuntu/Kubuntu 18.10
---

I have tried several times to use my Amazon Echo as a Speaker for my Home Office Setup. After all it's why I got it in the first place. However by default it would only connect as audio source which means I now get to hear Alexa's voice through my Laptop's speakers. Googling only gave me a few frustrated users and bug-reports. Here's what works for me:

## Delete Connections

Just to make sure we start from fresh.
1. Open the Alexa App and remove the connection to your Linux-Machine
1. Remove the connection from your Linux-Machine

## Connect from the Linux Machine

I could present some semi-solid theory why it is the case, but let's just cargo cult this: it's important to **connect from the Linux-Machine**.

1. Go to the Alexa App and start connecting to a new device.
2. From your Linux-Machine connect to your Echo. It seems your device has to be visible to do this. It may also fail a few times, just be stubborn and retry.

I had no luck using `bluetoothctl` so I used the wizard.

## Change profile to a2dp_sink

Open a terminal and enter `pactl list cards short`. You should see something similar to
```
0       alsa_card.pci-0000_00_1f.3      module-alsa-card.c
3       bluez_card.AA_BB_CC_DD_EE_FF    module-bluez5-device.c
```

Look for the card starting with `bluez_card`.

Now we need to change the active profile of our card:

```
pactl set-card-profile bluez_card.AA_BB_CC_DD_EE_FF a2dp_sink
```

Afterwards I had to change the default sink back to my internal speakers and then to the Echo again.

## It works

:tada:


## The catch

While this survives a re-connect, you'll have to do this after each reboot  ¯\\\_(ツ)\_/¯

Maybe I'll try to automate that some day...
