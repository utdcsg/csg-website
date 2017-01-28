---
layout: post
title: 'IFSF CTF #10 - Memories of never ending pain'
date: 2012-02-13 21:33:43.000000000 -05:00
type: post
published: true
status: publish
categories:
- Writeups
tags:
- exploitation
- ifsf
- mitch
- reverse engineering
- scott
meta:
  _edit_last: '1'
author:
  login: csgadmin
  email: utdcsg@gmail.com
  display_name: csgadmin
  first_name: ''
  last_name: ''
---

**Authors**: Mitchell Adair, Scott Hand

The problem starts out with the following text :

"We succeeded on gaining access to an old machine. We found an interesting binary that requests for an authorization code.
We believe that this code is still used somewhere else.

http://ctf.forbiddenbits.net/tasks/10.exe"

After opening the program in IDA, we realized it was an old-school DOS program. Installing DOS turned out to be a pain, so we ended up using a DOS emulator, DOSBOX (<http://www.dosbox.com/>). There are several version, so the debugging version is an import note.

To execute the program in DOSBOX you have to do 2 steps :
1. Mount your harddrive, "*mount c c:\\users\\myuser\\myproblemdir"*
2. Debug the program in debugging mode, "*debug 10.exe*"

So... after playing around with the program in debug mode, and looking at the program in IDA, there are a few major components that make up this program.

Three main function calls that represent the control flow of the program:

[<img src="%7B%7B%20site.baseurl%20%7D%7D/assets/debug-pic.png" title="IFSF Pic1" class="alignnone size-full wp-image-121" width="825" height="772" />](https://csg.utdallas.edu/wp-content/uploads/2012/08/debug-pic.png)

Looking a little more carefully at how the user input gets stored we see a little is applied to each character, before it is finally stored in \[si\]:

[<img src="%7B%7B%20site.baseurl%20%7D%7D/assets/store-user-input.png" title="IFSF Pic2" class="alignnone size-full wp-image-124" width="732" height="772" />](https://csg.utdallas.edu/wp-content/uploads/2012/08/store-user-input.png)

We can also see exactly where the altered user input is compared to a hard coded string in the program. If a value does match dx get's or'd with 5, essentially acting as a flag:

[<img src="%7B%7B%20site.baseurl%20%7D%7D/assets/key-comparison.png" title="IFSF Pic3" class="alignnone size-full wp-image-122" width="1465" height="993" />](https://csg.utdallas.edu/wp-content/uploads/2012/08/key-comparison.png)

Finally, we now know what happens to our user input, and we can see the final string we must match. A little python script can solve this problem :

[<img src="%7B%7B%20site.baseurl%20%7D%7D/assets/python-solution.png" title="IFSF Pic4" class="alignnone size-full wp-image-123" width="702" height="381" />](https://csg.utdallas.edu/wp-content/uploads/2012/08/python-solution.png)

And the solution is : 7R0LO101O
