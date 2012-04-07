---
layout: post
title: "Migrating Outlook to a Ubuntu mailbox"
date: 2011-03-03 11:10
comments: true
categories: 
---

Wanted to migrate a load of emails and contacts from Outlook to my Ubuntu netbook. Turns out it's really simple.

1) Copy the outlook ost file to the netbook. The location of this can be found by looking at outlook's settings. Important: make sure outlook is closed first!

2) Convert the ost file to a pst. There's a windows tool for this - but I found it worked ok under wine. The program's called ost2pst

http://www.windowsreference.com/ms-exchange-server/how-to-convert-ost-to-pst-format-for-outlook/

3) Install the readpst program (sudo apt-get install readpst - on Ubuntu). Then it's a simple command to create a UNIX mailbox (one file for each folder)

readpst outlook.pst

4) Enjoy your windows mail in the mail client of your choice. Being a fan of terminal apps, I'm using Alpine ;-)