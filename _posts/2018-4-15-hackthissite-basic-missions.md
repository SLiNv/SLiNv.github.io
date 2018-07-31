---
layout: post
title: "Hack This Site Basic Missions Write-up"
excerpt: "Hack This Site Basic Missions Write-up (Few Spoilers)"
categories: [hacking-write-ups]
comments: true
image:
  feature: hackthissite_basic/header.jpg
  show: true
---

Before my write-up, I want to mention that I read some well-written write-ups/tutorials after solving the Basic Missions and they do a good job guiding you to the right direction without spoilers. And there are a ton of this kind of well-written write-ups out there, you should read some of them just too see how others attack the same problem and their angles.

[Basic Mission Tutorial 1-10 - HackThisSite](https://www.hackthissite.org/articles/read/943)

[1-10 Basic Mission guide, that is completely noob friendly - HackThisSite](https://www.hackthissite.org/articles/read/758)

&nbsp;

#### Basic 1

Like it says, it's "The Idiot Test". The password is hidden in HTML source code.

#### Basic 2

I'm sure when everyone reads it, he/she will notice that "he neglected to upload the password file..." At first, I dived right into the source code to try to find the script but there is nothing. So what does it mean to not have a password file? Turned out that there is not a catch!! NO trick! :joy:

So he has the script to compare your password and the password in the file. But the password file is not there, then what is he comparing to your password? Exactly, NOTHING! :joy:

#### Basic 3

Now Sam has uploaded the password file. Inspect the form and understand what the process of submitting a/this form. You need some HTML form knowledge.

#### Basic 4

Since there's a button to send the password to Sam, we should take a look at it anyway. Inspect the form and you will see a POST request to level4.php

```HTML
<form action="/missions/basic/4/level4.php" method="post">
    <input type="hidden" name="to" value="sam@hackthissite.org">
    <input type="submit" value="Send password to Sam">
</form>
```

Let's test if they have [CSRF](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)) Protection!

You can modify the form in your browser's Dev Tool or copy the form to the local machine to run it. Bingo!

Another way of doing it is -- JavaScript Injection -- suggested by DoS-man, which is the post I referred at the beginning. However, nowadays most browsers prohibit JS Injections from the address bar. But there are a lot of interesting bypasses discovered by smart ppl.

#### Basic 5

hmmm, same. Changed the "to" value of the form to send to my email.

#### Basic 6

I found it out by trying different random strings. "aaa" or "aaaaaaaaaaaa" is a good start just to check out what happens.

#### Basic 7

According to the hint, Sam put the password file under the same directory. Meanwhile, there is a script that returns the output of the cal command. To pass this level, a little UNIX command line knowledge is required. Think about this, everything you put in the box is passed to the command ```cal``` in the command line, then how do you start another command along with the ```cal``` command?

#### Basic 8

Hopefully, you have noticed the generated file by Sam's daughter is SHTML. This is strange, so look up SHTML.

POSTNOTE: Server-Side Includes are directives present on Web applications used to feed an HTML page with dynamic contents. We can use the ```exec``` directive to execute a shell command or even a program/script. Therefore the answer is ```<!--#exec cmd="ls .."```

#### Basic 9

> Network Security Sam is going down with the ship - he's determined to keep obscuring the password file, no matter how many times people manage to recover it. This time the file is saved in /var/www/hackthissite.org/html/missions/basic/9/.

>In the last level, however, in my attempt to limit people to using server side includes to display the directory listing to level 8 only, I have mistakenly screwed up somewhere.. there is a way to get the obscured level 9 password. See if you can figure out how...

>This level seems a lot trickier then it actually is, and it helps to have an understanding of how the script validates the user's input. The script finds the first occurrence of '<--', and looks to see what follows directly after it.

If you have figured out Basic 8 you can pass Basic 9! The admin allows file/directory traversal to other folders besides Basic 8's folder. So it is a permission configuration mistake.

#### Basic 10

This time if you try a random password, it will tell you that

> You are not authorized to view this page.

Notice that the error message is different from the previous levels which they tell you that your password is wrong. With that in mind, maybe we should check the header to see if any authentication information is sent within it. If you see something name "cookie(s)"...

Nowadays, it's hard to retrieve cookies using JavaScript. JS Injection may not work for you, so replaying a POST request with FireFox's Developer Tools should be the easiest way to go. On the other hand, Burp Suite is always a handy tool to use in this kind of scenario.

#### Basic 11 (logical vs technical)
This level tripped me for a while... I have to say when I found out the password, I put in the password with many resignations.

So back to the topic, Sam decided to make a music site. Unfortunately, he does not understand Apache. 

Refresh the page and you will see the song name change every time. Think of what they have in common? Put on your 2000 hat. Think of how a music site organizes their directories. Then you will see our old friend you have probably seem before -- Apache's "auto-index" page. 

After you go all the way into the directories, it's an auto-index page with no file. Does it really don't have files in it or the file is hidden? Learn about "directory-level configuration file" and you will know!

After you find the password file, here is the most tricky part, at least to me... You look and look and look and look over and over and over again. You will finally find out if you stare at it for long enough. It's a magical 3-D website page, stand up and look at it or lie down or whatever angle you can think of to look at it and read it 10000 times.

Good luck!
