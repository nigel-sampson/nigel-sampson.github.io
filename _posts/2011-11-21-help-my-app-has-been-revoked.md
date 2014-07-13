---
layout: post
title: Help my app has been revoked!
tags: windows-phone
---

A couple of days ago I got a really nasty surprise, I went to open [To Do Today][todo] to add something to remember to do later that day and was shocked to see the message "To Do Today has been revoked by Microsoft, please uninstall this app". 

Wondering what sort of rule I had broken to have this happen to me I raced to my email and saw that I'd received nothing from Microsoft. Browsing the marketplace on the phone and the web showed that To Do Today still looked like it was certified and available for purchase.

The shoe dropped when I tried to open [Left to Spend][l2s] and received the same message. Since I'd developed both of these apps neither was actually purchased through the marketplace, they had been side-loaded on to the phone. I quickly checked my Marketplace account and verified that my phone was still listed as "developer unlocked", but to test this I decided to redeploy the app to the phone. Surprisingly this failed with an error message informing me that my phone was not developer unlocked.

Pulling open the developer unlock tool I retried unlocking the phone to no effect. The solution ultimately involved deregistering the phone against my account and then registering it again. As soon as the phone was registered all previously failing apps loaded successfully.

My advice if you see this message is "don’t panic", especially if you haven't received any notifications from Microsoft. Most likely you've run into some weird issue like this.


[todo]: /windows-phone/to-do
[l2s]: /windows-phones/left-to-spend

