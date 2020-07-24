---
title: "Mapping Keyboard for Microphone Muting"
date: 2020-07-24T16:11:25+08:00
tags: ["tutorial"]
draft: false
---


I really like my new Dell Precision 5540, however there are three things I miss from my previous Thinkpad:
* The camera is located at the bottom of screen resulting in an annoying angle during Remote Conferences
* The "Home" key requires "Fn" key to be held down which makes copying entire lines awkward
* There is no dedicated Microphone Mute button which means that quick Mute on/off is difficult in Remote Conferences

I spent a long time trying to figure out how to create a work-around for muting/un-muting my microphone quickly.  There are a lot of articles that indicate [how to mute microphone](https://www.makeuseof.com/tag/disable-microphone-windows-10/) - but they require several clicks which is not condusive to quick on/off muting.

You can additionally find some people that point to the [Mic Mute](https://sourceforge.net/projects/micmute/) software from SourceForge.  This seemed like a good option as it enables you to create keybinding for turning Microphone on and off as well as provides a status indicator on current status.  The project hasn't been updated since 2016 but still seems to work for most people.  Unfortunately it seemed that this program was causing some echo issues during my calls (i.e. echo would come on when I launched Mic Mute and disappear when killed the app).  It could be the echo was coincedence but given the program had not been updated and what seemed like voice quality issues I decided to not use this App.

I was about to give up and just deal with my lack of a dedicated microphone muting button but then I found [this article](https://www.autohotkey.com/boards/viewtopic.php?t=56866) on the AutoHotKey forum.  Implementation was pretty easy including:
1. Install AutoHotKey (via Chocolatey of course)
2. Run the "Find Microphone" script to determine the ID of your local microphone
3. Update the "Mute Microphone" script to use the found ID
4. Put the "Mute Microphone" script in your startup folder so it is executed on computer startup

I ended up updating the "Mute Microphone" script to bind to the PrintScreen button and so far it has been working great.  It seems to just toggle native Windows mute/un-mute status which means I don't expect any issues with distortion due to pass-through processing, etc.  Additionally AutoHotKey seems pretty light-weight so do not expect any significant incremental processing load.

