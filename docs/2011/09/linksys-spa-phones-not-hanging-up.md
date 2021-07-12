# Linksys SPA phones not hanging up
We are currently in the process or rolling out a new phone system. However we were having issues that if the other party hung up then the phone you called from would recognise the hangup and display that the call was terminated however would leave the line open then start beeping after about 10 seconds. While a minor issue it was annoying for using the soft keys to log into queues as you would have to press 2 buttons (1 to log in then 1 to hang up).

After a bit of research I found forums.digium.com/viewtopic.php?f=1&t=15122 Dimitripietro’s solution works however if you do that then if a call fails then it just silently hangs up. I then found a solution that appears to work with no ill effects at [http://asteriskfaqs.org/2010/11/24/asterisk-users/spa942-on-speaker-phone-does-not-hang-up.html](http://asteriskfaqs.org/2010/11/24/asterisk-users/spa942-on-speaker-phone-does-not-hang-up.html) (first comment from Peder)

- Log into the web interface
- Set the interface to advanced admin
- In the region tabs change the Reorder Delay to 255