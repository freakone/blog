---
layout: post
title: Device ID in STM32 microcontrollers
---

Identifying devices is in many projects a necessity. From simple device description to more sophisticated appliances as USB serial naming, security keys, cryptography keys etc. There are many ways of implementing unique IDs into a microcontroller. From simple hardcoding it into the firmware, separately flashing info FLASH memory to random generating by the first run of the device. When it comes to STM32 MCU's there is also another, simpler and cleaner possibility. During the manufaturing process a 96-bit ID is encoded onto the microcontroller.

This, so called, Unique ID consists of 3 parts:

* X and Y coordinates on the wafer expressed in BCD format
* lot number
* wafer number

## Accessing Unique ID

For UID access, you need simply to read memory at specified address. Keep in mind that different MCU line has a different location of this data sector in memory. At the end of this post I'll summarize most of them.

Since STM32 is a 32-bit processor we must perform three read-outs with a 32-bit offset to get a full 92-bit ID. Of course, we can use only a part of them.

For example, STM32F0 processors starting address is ```0x1FFFF7AC```.
So, to read full UID we must read following addresses:

{% highlight c %}
#define ID1 (*(unsigned long *)0x1FFFF7AC)
#define ID2 (*(unsigned long *)0x1FFFF7B0)
#define ID3 (*(unsigned long *)0x1FFFF7B4)
{% endhighlight %}

We can also define starting address as the beginning of an array:

{% highlight c %}
unsigned long *id = (unsigned long *)0x1FFFF7AC;
id[0]
id[1]
id[2]
{% endhighlight %}

As ```unsigned long``` in STM32 is a 32-bit variable, accessing array by index results in ```index*32``` bit offset from the starting address.

## Starting addresses

Below you can find Unique ID starting addresses for most of STM32 microcontrollers.

|Device line|Starting address|
|----|----|
| F0, F3 | 0x1FFFF7AC |
| F1 | 0x1FFFF7E8 |
| F2, F4 | 0x1FFF7A10 |
| F7 | 0x1FF0F420 |
| L0 | 0x1FF80050 |
| L0, L1 Cat.1,Cat.2 | 0x1FF80050 |
| L1 Cat.3,Cat.4,Cat.5,Cat.6 | 0x1FF800D0 |

