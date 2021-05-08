---
layout: post
title:  "Building Your Own Trezor Hardware Wallet - Part 2"
date:   2018-01-23 17:12:04
keywords:
  - Trezor
  - crypto
  - cryptocurrency
  - hardware wallet
  - diy
  - open source
  - pcb
  - soldering
---

{% include image.html name="trezor-pcb-unpopulated.jpg" %}

Unpopulated PCBs and components from Digikey have arrived!
Now to assemble them.

{%- comment -%} 
{% include image.html name="trezor-pcb-20180116-1.jpg" %}
{% include image.html name="trezor-pcb-20180116-2.jpg" %}
{%- endcomment -%}

## Assembly

Before this, I had never hand soldered components as small as 0402 package size before.
I was a bit worried that I wouldn't be able to do it but it turned out to not be that bad at all if you take your time and have a steady hand.
You can even get away with quite a large chisel tip on your soldering iron if you plan out which components to place first.

{% include image.html name="trezor-pcb-2.jpg" %}

It is always best to start with the smallest components first, and the ones that are in the most cramped locations.
After you can work your way up to the larger and taller components.

{% include image.html name="trezor-pcb-3.jpg" %}

I am still waiting for the OLED displays to arrive but I have completed assembly of one board sans display.
Overall I am pleased with how it turned out.

{% include image.html name="trezor-pcb-no-display.jpg" %}

Next step is to flash the STM32 microcontroller with the Trezor firmware and test it! 
I will probably have to use some of those small test probe hooks to connect to the 6-pin SWD connector as a temporary measure to flash it initially.
After that the firmware can be upgraded via the USB port.

## Note about ordering PCBs

As a side note, if I ever order PCBs in the future I probably will not choose Seeedstudio Fusion PCB again.
The reason being that I was forced to pay an additional $22.02 CAD in duty charges before I could receive the shipment.

This isn't necessarily the fault of Seeedstudio, but be aware that it can happen and that when it does the duty charges can be stupidly high relative to the total value of the product being shipped.
Perhaps it was because of the chosen courier (DHL) or perhaps I was just unlucky this time.

If I had a chance to do it again I'd probably choose [OSH Park][oshpark] instead.
I have ordered prototype PCBs from them before and so far I've never been hit with duty charges during pick-up.
They are a bit more expensive than Seeedstudio, but for low quantity this isn't really an issue.
Plus you get their signiture purple solder mask so yay to that I guess? :neutral_face:

[oshpark]: https://oshpark.com/
