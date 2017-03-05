---
date: 2013-10-08 00:00:00-06:00
layout: post
title: 'CSAW 2013 Write-up: Jordan Wiens (Recon-3, 100pts)'
---

This is a write-up for the ‘Jordan Wiens’ Recon-3 (100pts), for CSAW 2013.

For starters our hint is: "The trail starts where the trail ended." Since I solved this recon last year, I headed over to [Jordan Wiens website](http://key.psifertex.com/ "Jordan Wiens website, CSAW 2012 key") where the CSAW 2012 key was located. (After much frustration, the judges announced that the problem wasn't setup properly/user error was disabling you from solving it. The solution was to purge the cache, hence why the images are in Google Chrome Incognito windows).

At Jordan Wiens Website we see:
[<img src="{{ site.baseurl }}/assets/Screen-Shot-2013-09-19-at-10.59.12-PM.png" alt="Jordan Wiens Recon, CSAW 2013" class="aligncenter size-full wp-image-409"   />](https://csg.utdallas.edu/wp-content/uploads/2013/10/Screen-Shot-2013-09-19-at-10.59.12-PM.png)

The only thing here of use to me is: "Michael Vario sure does some suspicious signs, hope he doesn't do me." If you search this string in Google (using exact phrase search, [Google Search Opperators](https://support.google.com/websearch/answer/136861?hl=en "Google Search Operators")) you come across [this Tweet](https://twitter.com/cinnamon_carter/status/354111975099342848 "cinnamon_carter, tweet to Michael Vario") from cinnamon\_carter ([@cinnamon\_carter](https://twitter.com/cinnamon_carter "cinnamon_carter twitter")):
[<img src="{{ site.baseurl }}/assets/Screen-Shot-2013-09-29-at-11.57.33-PM.png" alt="cinnamon_carter, tweet to Michael Vario" class="aligncenter size-full wp-image-410"   />](https://csg.utdallas.edu/wp-content/uploads/2013/10/Screen-Shot-2013-09-29-at-11.57.33-PM.png)

Assuming that Jordan Wiens has a PGP key, we search for it (any public key server will do), and get:
[<img src="{{ site.baseurl }}/assets/Screen-Shot-2013-09-19-at-10.54.18-PM.png" alt="CSAW 2013, Jordan Wiens pgp key" class="aligncenter size-full wp-image-411"   />](https://csg.utdallas.edu/wp-content/uploads/2013/10/Screen-Shot-2013-09-19-at-10.54.18-PM.png)

Here we open the first link, since he tells us "CSAW folks: getting warmer."
[<img src="{{ site.baseurl }}/assets/Screen-Shot-2013-09-19-at-10.55.52-PM.png" alt="CSAW 2013, Jordan Wiens pgp key" class="aligncenter size-full wp-image-412"   />](https://csg.utdallas.edu/wp-content/uploads/2013/10/Screen-Shot-2013-09-19-at-10.55.52-PM.png)

Since we had the clue, that we're getting warmer: I decided to base64 decode his pgp key, (since pgp keys are base64 encoded, using: [pgpkey.txt](http://csg.utdallas.edu/wp-content/uploads/2013/10/pgpkey.txt) as the input) `Zacks-MacBook-Pro:CSAW zack$ base64 -D pgpkey.txt > base64decode`, which gives us: base64decode.

Opening our base64decode in any hex editor, you see:
[<img src="{{ site.baseurl }}/assets/Screen-Shot-2013-09-29-at-11.44.23-PM.png" alt="CSAW 2013, Jordan Wiens pgp key decode" class="aligncenter size-full wp-image-414"   />](https://csg.utdallas.edu/wp-content/uploads/2013/10/Screen-Shot-2013-09-29-at-11.44.23-PM.png)

In the ASCII text I noticed “JFIF” which I know is a [Magic Number](http://www.garykessler.net/library/file_sigs.html "List of magic numbers from Gary Kessler") for a jpeg, (Thankfully jpeg files have a file header and file trailer), so I took the hex from “FFD8” - “FFD9” (the start and end of a jpeg file in hex) and saved the file as [magic.jpeg](https://csg.utdallas.edu/wp-content/uploads/2013/10/magic.jpeg "CSAW 2013, Jordan Wiens recon key"). Upon opening our new image, you get:
[<img src="{{ site.baseurl }}/assets/magic.jpeg" alt="CSAW 2013 recon key - Jordan Wiens" class="aligncenter size-full wp-image-415"   />](https://csg.utdallas.edu/wp-content/uploads/2013/10/magic.jpeg)

You get the flag, key{mvarioisnotmyhomeboy}.

/endwriteup
-Zack
