---
layout: post
title:  "Incorrect date in headers when migrating from GMail to Outlook"
date:   2012-12-02 12:00:00
categories: outlook gmail
---

I decided to move my emails over to Outlook.com from GMail. Unfortunately, this proved to be more painful I thought. Microsoft provides a seemingly tool called 'Trueswitch' which does exactly this, but it does not work with custom domains -- I wanted to move stuff from a Google Apps account to a Windows Live Domains account. I tried the obvious and seemingly easy method of kindly asking Outlook.com to pull my emails from GMail through POP3, but that caused problems I didn't expect. You should know that years ago I have migrated a few tens of thousands of emails from my private Google Mail account (@gmail.com) to my Google Apps account using POP3 as well. Interestingly, when GMail fetches an email using POP3, then it prepends the message with a new 'Received' date field equal to the date when the message is fetched. Therefore my emails looked like

```
	Received: by 10.42.178.196 with SMTP id bazbar456; Thu, 30 Jun 2011 04:10:06 -0700 (PDT)
    Received: by 10.103.247.15 with SMTP id foobar123; Wed, 27 May 2009 05:03:37 -0700 (PDT)
```

Unfortunately, Outlook.com was using the first 'Received', therefore all my emails from before June 2011 were dated to 30 Jun 2011. This obviously messes up the conversation view in Outlook (and it's very annoying in general). I cannot affect how GMail or Outlook.com handles the emails, so independent of who's the culprit here (opinions differ), I had to do some kind of processing between the two mailboxes.

Outlook.com does not have an IMAP interface, it's using the proprietary ActiveSync procotol. Most of the desktop email clients cannot handle AS, so I tried to use the desktop Outlook. As you figure, my plan (or hope?) was that the desktop client will handle the 'Received' date properly and after downloading the messages (with the correct header) from GMail I can simply upload them to Outlook.com. Well, that didn't work, as seemingly Outlook has the same problem as Outlook.com, and after fetching 3 GB, I was very disappointed. My download speed is only around 2 Mbits/s, so I wanted to avoid re-downloading my entire mailbox again (if I've had fast broadband I would probably have used Thunderbird, and convert the .mbox file to .eml and then to .pst, which Outlook can handle).

I decided to copy the 'Sent' field to the 'Received' field for all messages in the .pst file, as these two should be more or less equal, and the 'sent' date seemed to be correct. I used Redemption, which exposes the object model over Outlook .pst files.


Steps:

   * Install Redemption (developer version)
   * Create a new project in Visual Studio, reference Redemption (COM)
   * Set a password for your Outlook data file (right click on the mailbox, properties, advanced) -- this is important!
   * Close Outlook
   * [http://pastebin.com/1gU3eLBE](http://pastebin.com/1gU3eLBE)
   * Run
   * Open Outlook
   * Voila, in theory, the 'Received' timestamps should be fixed
   * You can upload the emails now using AS
