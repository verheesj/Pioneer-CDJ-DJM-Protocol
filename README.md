# Pioneer-CDJ-DJM-Protocol
Protocol for Pioneer DJ DJM Mixer &amp; Pioneer CDJs

## PACKET SUMMARY:
```
Port  Typ  Len  Info
50000 0x00  86  Broadcast, CDJ, sent post-boot (after 0x0a)
50000 0x02  92
50000 0x04  80  Broadcast, CDJ, sent post-post-boot (After 0x00), appears to contain extra byte with channel membership
50000 0x06  69  Broadcast, CDJ&DJM, contains MAC Address, channel, sent every 2s
50000 0x0a  79  Broadcast, CDJ&DJM, broadcast at start-up three times
--
50001 0x03  82  Broadacst, DJM, contains live channels for on-air indicator
50001 0x0a
50001 0x26
50001 0x27
50001 0x28 138  Broadcast, CDJ&DJM, contains information on beatkeeping
50001 0x29
50001 0x2a
--
50002 0x05  90  CDJ-to-DJM Request for data?
50002 0x06 234  DJM-to-CDJ List of mountpoints (following a 0x05 packet)
50002 0x0a
50002 0x0d 238  DJM-to-CDJ List of files on mountpoint (following a 0x06 packet)
50002 0x29
```
## DJM boot
```
50000 0x0a (79 Bytes)
	Broadcast to subnet
	Data from DJM:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 0a 00 44 4a 4d 2d  Qspt1WmJOL..DJM-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 01 00 25 02                                   ...%.

50000 0x00 (86 Bytes)
	Broadcast to subnet
	Sent after 0x0a's third send
	Data:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 00 00 44 4a 4d 2d  Qspt1WmJOL..DJM-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 01 00 2c 01 02 74 5e 1c 56 65 a8              ...,..t^.Ve.

50000 0x02 (92 Bytes)
	Broadcast to subnet
	Sent by DJM three times after 0x00 is done
	Data:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 02 00 44 4a 4d 2d  Qspt1WmJOL..DJM-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 01 00 32 c0 a8 00 5a 74 5e 1c 56 65 a8 21 01  ...2...Zt^.Ve.!.
		0030   02 01        
	Only the third to last byte changes (00 -> 01 -> 02) 
	Contains MAC address (here, 74 5e 1c 56 65 a8)
```

## CDJ Boot

### DHCP request

*If no DHCP answer, it picks a link-local IP address (based on the MAC address): 169.254.xxx.yyy, where xxx and yyy are the last two octets of the MAC address.
```
50000 0x0a (79 bytes)
	Broadcast
	Three identical broadcasts
	Data from :BA in ch2:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 0a 00 43 44 4a 2d  Qspt1WmJOL..CDJ-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 02 00 25 01                                   ...%.
	Data from :9F in ch3:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 0a 00 43 44 4a 2d  Qspt1WmJOL..CDJ-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 02 00 25 01                                   ...%.
	CDJs appear to set last byte = 01
	DJMs appear to set last byte = 02

50000 0x00 (86 bytes)
	Broadcast once (DJM was connected)
	Data:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 00 00 43 44 4a 2d  Qspt1WmJOL..CDJ-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 02 00 2c 01 01 74 5e 1c 35 f3 9f              ...,..t^.5..
	Note presence of CDJ MAC address (74 5e 1c 35 f3 9f)

50000 0x01 (89 bytes)
	DJM immediately replies with this packet to :BA on ch2:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 01 00 44 4a 4d 2d  Qspt1WmJOL..DJM-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 01 00 2f c0 a8 00 5a 74 5e 1c 56 65 a8 01     .../...Zt^.Ve..
	DJM immediately replies with this packet to :9F on ch3:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 01 00 44 4a 4d 2d  Qspt1WmJOL..DJM-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 01 00 2f c0 a8 00 5a 74 5e 1c 56 65 a8 01     .../...Zt^.Ve..
	Note presence of DJM MAC address (74 5e 1c 56 65 a8)

50000 0x04 (80 bytes)
	CDJ replies with broadcast containing channel number:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 04 00 43 44 4a 2d  Qspt1WmJOL..CDJ-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 02 00 26 03 01                                ...&..
```

## CDJ Boot when plugged into a switch

### DHCP discover
### DHCP request
```
50000 0x0a (79 bytes)
	Broadcast three times from CDJ
	CDJ :BA ch2 data:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 0a 00 43 44 4a 2d  Qspt1WmJOL..CDJ-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 02 00 25 01                                   ...%.

50000 0x00 (86 bytes)
	Broadcast three times from CDJ
	Byte 0x24 changes, [0x01, 0x02, 0x03] otherwise identical packets                       ..
		0000   51 73 70 74 31 57 6d 4a 4f 4c 00 00 43 44 4a 2d  Qspt1WmJOL..CDJ-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 02 00 2c 02 01 74 5e 1c 06 a4 ba              ...,..t^....
	Note presence of CDJ MAC address at end

50000 0x02 (92 bytes)
	Broadcast three times
	CDJ :BA ch2 data:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 02 00 43 44 4a 2d  Qspt1WmJOL..CDJ-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 02 00 32 c0 a8 00 5c 74 5e 1c 06 a4 ba 02 0x  ...2...\t^......
		0030   01 02                                            ..
	Note presence of CDJ MAC address
	MAC address followed by four bytes, including anther incrementing number (0x here)
```

Now we see this:
```
50002 0x05 (80 bytes)
	From DJM to CDJ:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 05 00 44 4a 4d 2d  Qspt1WmJOL..DJM-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 01 00 26 21 01                                ...&!.
```

## CDJ boot onto allocated channel
```
50000 0x08 (83 bytes)
	CDJ3 to CDJ.new:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 08 00 43 44 4a 2d  Qspt1WmJOL..CDJ-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 02 00 29 03 c0 a8 00 5d                       ...)....]
	CDJ2 to CDJ.new:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 08 00 43 44 4a 2d  Qspt1WmJOL..CDJ-
		0010   32 30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00  2000nexus.......
		0020   01 02 00 29 02 c0 a8 00 5c                       ...)....\
	Appears to be the equivelant of "Hey jackass, I'm using this channel, get your own"
```

## Master/slave assignment test cases
```
Boot CD3 on. Play CD. Assign master by pressing button.
Boot CD2 on.
Assign CD2 as master from mixer.

Boot CD3 on. Play CD. Assign master by pressing button.
Boot CD2 on.
Assign CD2 as master from CD2 master button.

Boot CD3 on. Play CD. Assign master by pressing button.
Boot CD2 on. Play file.
Assign CD2 as master by sliding fader.

Boot CD3 on. Play CD. Assign master by pressing button.
Boot CD2 on. Play file.
Assign CD2 as master by pressing CDJ3 platter.

Boot CD3 on. Play CD. Assign master by pressing button.
Boot CD2 on. Play file.
Assign CD2 as master by reversing CDJ3 playback direction.

Boot CD3 on. Play CD. Assign master by pressing button.
Boot CD2 on. Play CD.
Assign CD2 as master by pressing CD3 master button.
```
## "Link" functionality
```
50002 0x05 (90 Bytes)
	From CDJ to DJM
	Appears to be a request for mountpoints
	Example:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 05 43 44 4a 2d 32  Qspt1WmJOL.CDJ-2
		0010   30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00 01  000nexus........
		0020   00 02 00 0c c0 a8 00 5c 00 00 00 21 00 00 00 05  .......\...!....

50002 0x06 (234 bytes)
	From DJM to CDJ
	Appears to be a list of mountpoints (i.e. "[Mixer icon] DJM-2000")
	Data:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 06 44 4a 4d 2d 32  Qspt1WmJOL.DJM-2
		0010   30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00 01  000nexus........
		0020   01 21 9c 00 00 00 00 21 00 00 00 05 00 44 00 4a  .!.....!.....D.J
		0030   00 4d 00 2d 00 32 00 30 00 30 00 30 00 00 00 00  .M.-.2.0.0.0....
		0040   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
		0050   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
		0060   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
		0070   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
		0080   00 00 00 00 00 56 00 65 00 72 00 2e 00 30 00 2e  .....V.e.r...0..
		0090   00 30 00 30 00 00 00 00 00 00 00 00 00 00 00 00  .0.0............
		00a0   00 00 00 00 00 00 00 01 00 00 00 00 00 00 00 00  ................
		00b0   00 00 00 01 00 00 00 01 00 00 00 00 00 00 00 00  ................

50002 0x0d (238 bytes)
	From DJM to CDJ
	Appears to be a list of files on mountpoint
	Data:
		0000   51 73 70 74 31 57 6d 4a 4f 4c 0d 44 4a 4d 2d 32  Qspt1WmJOL.DJM-2
		0010   30 30 30 6e 65 78 75 73 00 00 00 00 00 00 00 01  000nexus........
		0020   01 21 e0 ff 00 00 00 21 00 00 00 01 00 44 00 4a  .!.....!.....D.J
		0030   00 4d 00 46 00 49 00 4c 00 45 00 2e 00 57 00 41  .M.F.I.L.E...W.A
		0040   00 56 00 00 00 00 00 00 00 00 00 00 00 00 00 00  .V..............
		0050   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
		0060   00 00 00 00 00 00 00 00 00 00 00 00 00 43 00 3a  .............C.:
		0070   00 2f 00 44 00 4a 00 4d 00 46 00 49 00 4c 00 45  ./.D.J.M.F.I.L.E
		0080   00 2e 00 57 00 41 00 56 00 00 00 00 00 00 00 00  ...W.A.V........
		0090   00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
		00a0   00 00 00 00 00 00 00 00 00 00 00 00 00 30 00 30  .............0.0
		00b0   00 3a 00 30 00 33 00 00 00 31 00 35 00 33 00 36  .:.0.3...1.5.3.6
		00c0   00 00 00 00                                      ....

50001 0x28
	Sent by all devices when they generate a beat, once per beat.
	Also contains info on BPM and timing of next beat.
	

packet4d==04:
	40 bytes (82 total)
	Sent by all devices, for device discovery?

packet4d==36
	54 bytes (96 total)

	Format:
	51:73:70:74:31:57:6d:4a 4f:4c:06:00:43:44:4a:2d
	32:30:30:30:6e:65:78:75 73:00:00:00:00:00:00:00
	01:02:00:36:02:02:[device_MAC_addr]:c0:a8:00:72
	03:00:00:00:01:00



packet4d==3c MIDI Beat packet
	196 bytes (138 total)
	Contains timecode, once every beat it appears, at byte 86
```

CDJ startup sequence with no devices on network:

```
1. Sends out a DHCP Discover request up to three times. Wait 1s, 2s, then 3s.
2. If no DHCP reply, fall back to link-local address negotiation. Send out three ARP requests at 1s intervals.
3. Sends packetType=x0a 3 times, 300ms wait
4. Sends packetType=x00 3 times, 300ms wait
5. Sends packetType=x02 3 times, 300ms wait
6. Sends packetType=x04 3 times, 300ms wait
6. Sends packetType=x06 packet, 400ms wait
7. Begins regular broadcast of packetType=x06 packets, 1s intervals
```

CDJ startup sequence with mixer on network:
```
1. Sends out a DHCP Discover request up to three times. Wait 1s, 2s, then 3s.
2. If no DHCP reply, fall back to link-local address negotiation. Send out three ARP requests at 1s intervals.
3. Sends packetType=x0a 3 times, 300ms wait
4. Sends packetType=x00 3 times, 300ms wait (If the device is plugged directly into the mixer, the mixer will reply to it here)
5. Sends packetType=x02 3 times, 300ms wait
6. Sends packetType=x04
7. DJM replies packetType=x05, waits 180ms
8. DJM sends packetType=2_x29 at 200ms intervals
9. Meanwhile, CDJ sends packetType=x06, waits 1300ms
10 Sends packetType=x06 at 200ms intervals
```
The packetType=x06 heartbeat/discover packets emitted at 2-second intervals appear to tell other devices about the existance of a mountable filesystem. On a network with one DJM and one CDJ, after the DJM broadcasts this packet, the CDJ attempts to mount the DJM.

Information about the current track of another device (Link Info) seems to be provided by remote devices to local TCP port 1054

CDJ track information TCP server test:
```
A. Boot up CDJ3
1. Turn on CDJ2 with CD inside, waiting until CD metadata appears on screen
2. Hold "Link INfo button" on CDJ3, wait until CDJ2 metadata appears on screen
3. Kill sniffer
```
