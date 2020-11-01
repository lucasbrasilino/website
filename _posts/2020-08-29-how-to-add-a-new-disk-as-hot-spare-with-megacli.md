---
title: "How to add a new disk as hot spare with MegaCLI"
date: 2020-08-29 14:50:00 -0500
categories: [Sysadmin, RAID]
tags: [raid,megacli,megaraid]
---

As you might now, MegaCLI is the _command line interface_ application for
managing LSI MegaRAID family of hardware RAID controllers. It is very powerful,
yet many times cumbersome, due to its award set of flags and options.
You can find a large number of documentation about MegaCLI, from an 
[official manual](https://www.supermicro.com/manuals/other/MegaRAID_SAS_Software_Rev_I_UG.pdf){:target="_blank"} to
[cheat sheets](http://erikimh.com/megacli-cheatsheet/){:target="_blank"} and
[wikis](https://wikitech.wikimedia.org/wiki/MegaCli){:target="_blank"}.

I've seen plenty of information on how to create new arrays, with or without
hot spare drives. However I haven't find any that says what to do when you _add_ a
new disk to a enclosure, how to make it a hot spare for a _preexistent_
array. This is what this post is about. I'll tell you, it was a little tricky.

Before we begin, you should be familiar with the terms **enclosure** and
**slot**. Enclosure is a chassi where disks are installed. They are enumerated
because a single MegaRAID adapter may controls a number of them. Slot is simply a
space in the enclosure where you physically install a disk. Thus, the slot ID is
used interchangeably as a disk ID. You will see all the time in documentation
the identifier `[E:S]`. This is simply referencing to a disk in slot `S`, installed
in the enclosure `E`. 

All shell commands here were executed as `root`, indicated by the prompt symbol
`#`. In the system I am working on the enclosure ID is `252`.

Let's           say           we          have           a           preexistent
 [RAID-5](https://en.wikipedia.org/wiki/Standard_RAID_levels#RAID_5){:target="_blank"}
 with 6 disks (from slot `0` to `5`), and you just installed a brand new disk in
 slot `6`.  By the way, good  practices suggest that  the new disk should have the same
 model/capacity of  others. If not possible, the  new disk must  have at
 least the same capacity of the largest one.

Once   you   connected   the   new   disk    it   will   be   available   as   a
[**JBOD**](https://en.wikipedia.org/wiki/Non-RAID_drive_architectures#JBOD){:target="_blank"}
disk. This  means it  has no  relationship with  any other  RAID in  the system.
Using the following command you can see the current `Firmware state` property:
 
```
# megacli -PDInfo -PhysDrv [252:6] -a0
                                     
Enclosure Device ID: 252
Slot Number: 6
[....]
PD Type: SAS

Raw Size: 1.819 TB [0xe8e088b0 Sectors]
Non Coerced Size: 1.818 TB [0xe8d088b0 Sectors]
Coerced Size: 1.818 TB [0xe8d00000 Sectors]
Sector Size:  512
Logical Sector Size:  512
Physical Sector Size:  512
Firmware state: JBOD
Port's Linkspeed: 6.0Gb/s 
[...]
Drive has flagged a S.M.A.R.T alert : No
```

Surprisingly enough, in my experimentations the controller does not apply any
further configuration on a disk with a state `JBOD`. That is annoying
because JBOD term alludes that disk may used as a standalone disk, and since disk
is not marked as **bad** you could use it right away.

So,  here comes  the trick.  You  might change the  firmware state  to
`Unconfigured-Good` using the `-PDMakeGood` option. After the controller set
disk as good, it spins disk up making the disk ready for any usage.

```
# megacli -PDMakeGood -PhysDrv [252:6] -Force -a0
                                     
Adapter: 0: EnclId-252 SlotId-6 state changed to Unconfigured-Good.

Exit Code: 0x00
root@kanar:~# megacli -PDInfo -PhysDrv [252:6] -a0
                                     
Enclosure Device ID: 252
Slot Number: 6
[...]
PD Type: SAS

Raw Size: 1.819 TB [0xe8e088b0 Sectors]
Non Coerced Size: 1.818 TB [0xe8d088b0 Sectors]
Coerced Size: 1.818 TB [0xe8d00000 Sectors]
Sector Size:  512
Logical Sector Size:  512
Physical Sector Size:  512
Firmware state: Unconfigured(good), Spun Up
[...]

Exit Code: 0x00
```

`Exit Code: 0x00` means the operation was completed successfuly.

Finally, you can set the new disk as hot spare. In this example, it will be a
**global** hot spare, meaning that it will replace faulty disk from _any_
RAID. MegaRAID also supports **dedicated** hot spare, which as it sounds is
dedicated to a particular array.

```
# megacli  -PDHSP -Set -PhysDrv [252:6] -a0
                                     
Adapter: 0: Set Physical Drive at EnclId-252 SlotId-6 as Hot Spare Success.

Exit Code: 0x00
```

At the end, you can check the new disk hot spare status:

```
# megacli -PDInfo -PhysDrv [252:6] -a0
                                     
Enclosure Device ID: 252
Slot Number: 6
[...]
PD Type: SAS
Hotspare Information: 
Type: Global, is revertible

Raw Size: 1.819 TB [0xe8e088b0 Sectors]
Non Coerced Size: 1.818 TB [0xe8d088b0 Sectors]
Coerced Size: 1.818 TB [0xe8d00000 Sectors]
Sector Size:  512
Logical Sector Size:  512
Physical Sector Size:  512
Firmware state: Hotspare, Spun Up
[...]
Port's Linkspeed: 6.0Gb/s 
Drive has flagged a S.M.A.R.T alert : No

Exit Code: 0x00
```

Note the `Type: Global, is revertible` property. Being _reversible_ means that
in the case the hot spare is active with a copy of a faulty disk, when you
replace the faulty disk, the controller will copy data from the hot spare to the
new disk and will make the hot spare be back to its original _hot spare
status_.

In this post we discussed a simple, but not very documented, way to configure a
newly added disk as a hot spare on a MegaRAID controller using MegaCLI.

Bye for now!

