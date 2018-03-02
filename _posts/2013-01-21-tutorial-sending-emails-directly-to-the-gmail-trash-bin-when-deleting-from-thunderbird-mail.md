---
id: 388
title: 'TUTORIAL: Sending emails directly to the Gmail trash bin when deleting from thunderbird mail'
date: 2013-01-21T19:56:18+00:00
author: Robert
layout: post
guid: http://wiredtron.com/?p=388
permalink: /2013/01/21/tutorial-sending-emails-directly-to-the-gmail-trash-bin-when-deleting-from-thunderbird-mail/
categories:
  - Linux
---
<a href="http://wiredtron.com/2013/01/21/tutorial-sending-emails-directly-to-the-gmail-trash-bin-when-deleting-from-thunderbird-mail/screenshot-from-2013-01-21-214122/" rel="attachment wp-att-389"><img class="alignright size-medium wp-image-389" alt="Thunderbird version" src="http://wiredtron.com/wp-content/uploads/2013/01/Screenshot-from-2013-01-21-214122-300x158.png" width="300" height="158" srcset="https://wiredtron.com/wp-content/uploads/2013/01/Screenshot-from-2013-01-21-214122-300x158.png 300w, https://wiredtron.com/wp-content/uploads/2013/01/Screenshot-from-2013-01-21-214122.png 625w" sizes="(max-width: 300px) 100vw, 300px" /></a>
  
I just recently started using Ubuntu again, and one of the biggest challenges I have ran into has been with the default mail client, Thunderbird. This guide explains how to fix one of the biggest problems I have been encountering with thunderbird, sending emails directly to the trash bin without having thunderbird create a nasty [imap]/trash bin.

**Background:** The way thunderbird decides which folder to store your deleted email in is under a setting, which can be found on Ubuntu by going to Edit > Account Settings. If you then browse to the server settings section you will see a label which says &#8220;When I delete a message, move to this folder:&#8221; and then a trash folder. The problem is this is really referring to the [imap]/Trash folder! Hit the cancel button on the Account Settings window for now.

**How to fix:** The reason the Gmail trash folder is not listed under Server Settings is because the gmail trash folder is not subscribed to by default. You can fix this by right clicking on your [Gmail] folder in thunderbird and then clicking on &#8220;Subscribe&#8230;&#8221;

The window that pops up will have a [Gmail] folder listed at the bottom which will be collapsed. If you expand it you will notice something, the Trash folder is unchecked! You will need to then check the checkbox next to the Trash folder and hit Ok.

<a href="http://wiredtron.com/2013/01/21/tutorial-sending-emails-directly-to-the-gmail-trash-bin-when-deleting-from-thunderbird-mail/screenshot-from-2013-01-21-215018/" rel="attachment wp-att-390"><img class="alignleft size-medium wp-image-390" alt="Screenshot from 2013-01-21 21:50:18" src="http://wiredtron.com/wp-content/uploads/2013/01/Screenshot-from-2013-01-21-215018-300x116.png" width="300" height="116" srcset="https://wiredtron.com/wp-content/uploads/2013/01/Screenshot-from-2013-01-21-215018-300x116.png 300w, https://wiredtron.com/wp-content/uploads/2013/01/Screenshot-from-2013-01-21-215018.png 535w" sizes="(max-width: 300px) 100vw, 300px" /></a>

&nbsp;

&nbsp;

&nbsp;

&nbsp;

Once you have added this option, then proceed back to Edit > Account Settings and browse to the Server Settings section. Under when I delete a message, choose Move it to this folder > Click on the dropdown and then mouse over the [Gmail] folder, and choose Trash. Make sure you choose the Trash bin that is now available in the [Gmail] folder, not the local one! If you choose the local one you will be in the same boat you were in previously.

Now find an email you want to delete and check in your Gmail trash, it will be there!