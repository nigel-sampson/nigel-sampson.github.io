---
layout: post
title: Failing Silently and the Contact page
tags: 
---

It&#39;s come to my attention that the Contact page has been failing silently. The issue was around sending the actual email which the mail server had started timing out on rather than sending. The worst part of all was that default code for the contact page in Blog Engine doesn&#39;t do anything about it.&nbsp; The actual SmtpException is caught and reported through an event, where as the UI code waits for an exception to report a failed send.

My apologises if you&#39;ve tried to contact me lately (I&#39;m uncertain how far back this behaviour goes). It should be working correctly now.

Thanks 


