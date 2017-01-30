---
layout: post
title: 'CSAW 2013 Write-up: historypeats (Recon-5, 100pts)'
date: 2013-10-18 19:55:03.000000000 -04:00
type: post
published: true
status: publish
categories:
- Writeups
tags: []
meta:
  _edit_last: '1'
author:
  login: csgadmin
  email: utdcsg@gmail.com
  display_name: csgadmin
  first_name: ''
  last_name: ''
---

This is a write-up for the “historypeats’ Recon-5 (100pts), for CSAW 2013.

Our initial hint is a Google search for “historypeats.” Completing the search we get a
small number of results. The second result of the search is his [GitHub page](https://github.com/historypeats "Github: historypeats"). The contribution activity revealed some new changes so if we look in the public activity
section. We can see he added and removed some comments.

[<img src="%7B%7B%20site.baseurl%20%7D%7D/assets/historypeats1.png" alt="historypeats_github" class="aligncenter size-full wp-image-460" width="1432" height="774" />](https://csg.utdallas.edu/wp-content/uploads/2013/10/historypeats1.png)

If we click on the link we get:

[<img src="%7B%7B%20site.baseurl%20%7D%7D/assets/historypeats2.png" alt="historypeats2_github" class="aligncenter size-full wp-image-461" width="1920" height="1041" />](https://csg.utdallas.edu/wp-content/uploads/2013/10/historypeats2.png)

And there is our flag, key{whatDidtheF0xSay?}.

/endwriteup
All props to Solomon.
-Zack
