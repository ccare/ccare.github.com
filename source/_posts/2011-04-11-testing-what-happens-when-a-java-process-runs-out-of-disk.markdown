---
layout: post
title: "Testing what happens when a java process runs out of disk"
date: 2011-04-11 12:32
comments: true
categories:
- testing
- java
---

Needed to do some smoke testing of a java app that was downloading files. In particular I wanted to simulate it running out of disk.

I didn't want to spin up a virtual machine, so instead I created a virtual disk and mounted it as an ordinary directory. I'd forgotten which commands to use to create, format, and mount a loop device. Blogging them so I can find them again.

```bash
# Create a raw 'disk' around 1 GB in size
dd if=/dev/zero of=disk.raw bs=1k count=1000000
# Format it - I went for ~900MB which suited my tests
mkfs -t ext2 disk1.raw  900000
# Mount the file into a local folder
sudo mount -o loop -t ext2 disk.raw ./mount-dir
```

Works a treat, and really easy to play with different disk sizes.