---
layout: post
title: "Extracting calendar entries from Outlook"
date: 2011-03-03 12:06
comments: true
categories:
- Migration
---

Having migrated by Outlook mailbox, I wanted to load calendar entries into another calendar. Following my earlier post, I already had my calendar entries in ICAL format, so all seemed good. However, I also had a number of recurring events (birthdays etc) that I wanted to put into my Google Calendar. The following PERL script was a handy bit of code to extract the recurring events so that I could extract them and add to my public calendar.

```bash
$entry =  '';
while(<STDIN>) {
  if ($_ eq "" || $_ eq "\n") {
    if ($entry =~ /^RRULE\:/) {
      print $entry;
    } 
    $entry = "\n";
  } else {  
     $entry .= $_;
  }
}
print $entry;
```