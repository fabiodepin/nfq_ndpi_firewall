# NFDPI
It is a user-space deep packet inspection tool capable of detecting and blocking of 227 application protocols.

## Why
So far there is a lack of open-source DPI tools that allow users to block packets coming from specific application.
L7filter is outdated, openappid is slow and works only with Snort, so I created this one.

## Before you start
nfdpi was created specificaly for a custom Linux-based OS with a patched core, therefore currently there are limitations when running on distributions with standard Linux core. Read [Limitations](#limitations) for details.

This program is a PoC, so don't expect too much.

## Features
1. Processing packets from multiple queues (one queue per thread)
2. Detection of 227 supported protocols (full list can be found on [nDPI web page](http://www.ntop.org/products/deep-packet-inspection/ndpi/)).
3. Labeling connections for which the protocol detection has ended in Linux's connection tracking subsystem.
4. Periodic memory cleanup based on time flows have been idle.
5. Fully configurable parameters from the command line.
6. Printing packet header information and protocol to screen.

## Requirements
- nDPI 2.8
- libnetfilter_queue
- libnetfilter_conntrack

## Installation
- install libnetfilter_queue and libnetfilter_conntrack (for Ubuntu should be possible with `apt-get install`)
- `git clone https://github.com/robingoth/nfq_ndpi_firewall.git`
- `cd nfq_ndpi_firewall/lib`
- download and compile nDPI like described [here](https://github.com/ntop/nDPI/blob/2.0-stable/INSTALL)
- `cd /path-to-ndpi-nfq-firewall/src/`
- `make`

## Usage
You can start with running `./ndpi --help`.
Normally you should start with copying [connlabel.conf](./connlabel.conf) file from this repository into `/etc/xtables/`.
Then there are two options you have: single-queue and multi-queue.

### Single queue
1. Run `./nfdpi`
2. In another terminal run `iptables -I FORWARD -m connlabel ! --label NDPI_DETECTION_OVER -j NFQUEUE --queue-num 10`. I'm not going to explain how NFQ and CONNTRACK work, but the above command means "enqueue all packets from FORWARD chain that don't have NDPI_DETECTION_OVER label on a connection that they belong to".

### Multiple queues (4)
1. Run `./ndpi -q 10 -q 11 -q 12 -q 13`
2. In another terminal run `iptables -I FORWARD -m connlabel ! --label NDPI_DETECTION_OVER -j NFQUEUE --queue-balance 10:13`

### Manipulating packets
After you have launched the program nfdpi will start labeling connections in connection tracking subsystem of Linux.

For example, you can run `iptables -I FORWARD -m connlabel --label FACEBOOK -j DROP` in case you want to drop packets coming from facebook web site. You can substitute "FACEBOOK" with any other protocol from your *connlabel.conf* file.

## How does it work
![alt text](https://github.com/robingoth/nfq_ndpi_firewall/blob/master/docs/workflow.png "nfdpi Workflow")

You can also read a [chapter of my thesis](./docs/thesis_chapter_l7firewall.pdf) for more details.

## Limitations
**CONNTRACK** supports only 128 unique labels, so for now nfdpi can label only 128 first protocols from [nDPI source files](https://github.com/ntop/nDPI/blob/2.0-stable/src/include/ndpi_protocol_ids.h).

If you want to increase this number you have to patch your Linux core by changing *XT CONNLABEL MAXBIT* value in [include/uapi/linux/netfilter/xt_connlabel.h](https://github.com/torvalds/linux/blob/master/include/uapi/linux/netfilter/xt_connlabel.h) file to a higher one.

## Future work/features
- Automated tests.
- Recover NFQ socket connection in case it fails.
- Possibility to choose a subset of protocols to be labeled. Instead of 128 first protocols from ndpi sources user should be able to choose any 128 protocols form the list.

