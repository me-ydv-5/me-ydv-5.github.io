---
layout: post
title:  "GSoC Summary"
permalink: /gsoc2018/summary
comments: true
date:   2018-08-07 17:48:00 +0530
categories: jekyll update
---

# GSoC 2018, GNU Octave
## Octave Code Sharing - Summary

****

Now that the coding period has finally ended, I'm happy to share the summary of my project.

My project was Octave Code Sharing. As the name suggests, it mainly focused (the first half of the project) on "sharing" the Octave code. The intended design was, the user should be able to push their octave script onto [wiki.octave.org](https://wiki.octave.org) for others to use and see. 

For this, I needed a script to produce the script in a format that is acceptable by MediaWiki. My mentor had already done this bit by implementing the [`publish`](https://bitbucket.org/me_ydv_5/octave/src/default/scripts/miscellaneous/publish.m) function. What this does is, it takes the user script and converts it to the prescribed format (if available), by default being `html`. So to convert it to wiki format, we needed [__publish_wiki_output\_\_.m](https://bitbucket.org/me_ydv_5/octave/src/default/scripts/miscellaneous/private/__publish_wiki_output__.m), which I refactored a bit. This file is responsible for producing the wiki output. `publish` is used to parse the script file. After this, the main task was to find a way to upload the formatted file to the wiki server. An excellent reference point for getting started with this was [this script](https://github.com/octave-de/OctConf2017/blob/master/demo2/wikiLogin.sh). The script was written by the mentor as a bash script. So, I had to convert this script into an Octave counterpart so that we won't need to execute third party code (bash script in this case) to transfer the content of the file to the wiki server.

Because the backend of Octave is mostly C++, I needed to use [libcurl](https://curl.haxx.se/libcurl/) library to perform the actual transfer, because Octave does not have its own implementation of such a library and many other areas of the software use libcurl as well.

The workflow is like this:
1.  Input the script, user's password and username to [`publish_to_wiki`](https://bitbucket.org/me_ydv_5/octave/src/default/scripts/miscellaneous/publish_to_wiki.m) function, which converts the script to wiki format using `publish` function internally and saves the output file to a directory named `html` (analogous to MATLAB).
2.  The same function then picks out the figures from the script, if any.
3.  Then it inputs the figures, formatted output file content and credentials to [`wiki_upload`](https://bitbucket.org/me_ydv_5/octave/src/default/scripts/miscellaneous/wiki_upload.m) script.
4.  `wiki_upload` script then establishes the connection with the wiki server by asking for a login token.
5.  It stores the cookies in a temporary file.
6.  CSRF Login is then performed to the wiki server and an edit token (for editing a page) is obtained.
7.  The wiki formatted output file from `publish` is then uploaded to the server.
8.  Figures are uploaded at last, with proper verification such that it doesn't upload a particular figure if it alreadty exists on the server.

To test yourself, you may want to use the following command:

`publish_to_wiki ("script_file", "username", "password")`

This should place your script on a URL similar to `https://wiki.octave.org/script_file`.

An already performed example of this is, when I tried to publish a script named [intro.m](https://github.com/octave-de/OctConf2017/blob/master/demo2/intro.m) on the [test wiki server](https://wiki.octave.space) set up by the mentor.
I used the command, 

```
publish_to_wiki("intro", "myUserName", "myPassword");
```

and I got [this page](https://wiki.octave.space/index.php/Intro) as the result. This page has *not* been written manually on the server, rather its all about the function.

An important point in this process is the storing of cookies, which was performed by libcurl and set up in [`liboctave/url-transfer.h`](https://bitbucket.org/me_ydv_5/octave/src/aa660b7dcae72a2e769ecbfe71b56c86012ca2db/liboctave/util/url-transfer.h#lines-158). The most challenging part in this part of the project was to figure out the uploading of figures, which I was able to do in a week or so. This needed me to read a lot of documentation and codes. Finally, the figures were uploaded using the [`form_data_post`](https://bitbucket.org/me_ydv_5/octave/src/aa660b7dcae72a2e769ecbfe71b56c86012ca2db/liboctave/util/url-transfer.cc#lines-721) function.

This was the first half part of the project.

Next was implementing the MATLAB compatible [RESTful services for Octave](https://github.com/octave-de/octave-web#the-intended-design). The project included implementing the [`weboptions`](https://bitbucket.org/me_ydv_5/octave/src/ocs/scripts/web/weboptions.m), [`webread`](https://bitbucket.org/me_ydv_5/octave/src/ocs/scripts/web/webread.m) and [`webwrite`](https://bitbucket.org/me_ydv_5/octave/src/ocs/scripts/web/webwrite.m) functions. Other implementations of the suite will be followed in post GSoC work.

In this part of the project, working on `weboptions` was very fun. I got to learn a lot about the internal processing of function files and how Octave uses handle classes to cater the needs of user defined classes. Also, the way getters and setters work is really nice, even though there is still space for Classdef in octave.

`weboptions` is used to put header fields in the other two functions.

`webwrite` is used to POST something to a URI. It can also take a weboptions object as an argument to amend the header fields.

`webread` is used to GET something from a URI. It also accepts weboptions object as an input.

You can refer to the help text of the functions to know more about them. A few of the weboptions functions are not yet implemented, which are listed in the help text, so they won't work with webwrite and webread either.

The challenging part in this part of the project was, passing the weboptions object from the octave script function to the C++ backend and then performing the required operations on them. These functions, too, use `libcurl` for sending HTTP requests.

All the goals of the project were met. There were times when I was unable to get the thing done, be it trivial or not, but with continuous lookup and exploring, I was able to get them done. Sometimes, there are reasons that are out of your reach, like the sudden power failures that I encountered in my machine during the last phase. So, Kai, my mentor, being at his best, understood the concern and extended the weekly plan by a couple of days after I requested. I had also implemented a [test server](https://bitbucket.org/me_ydv_5/server_code) in java for GET and POST request only to find a week later that there's one easy alternative of [https://httpbin.org/get](https://httpbin.org/get) and [https://httpbin.org/post](https://httpbin.org/post), respectively.

## Further work after GSoC is over.

As for the `publish_to_wiki` function, all the tests that I did were as desired, however if any error is reported it will be sorted out then and there.

There are a few options left in the `weboptions` which are not currently implemented (this was intended). It would be extended as well using `jsonencode` and `jsondecode`, essential for the next round of renovation in this function.

`webwrite` works pretty good in case of sending HTTP requests in text form, but it is still unable to resolve a query list of JSON form. That is one thing that's on my to-dos.

I'm currently trying to polish `webread`, although it is in workable state, but still there's space for some improvement. For example, when I try to GET an image from a URI, I am unable to decode the binary data from the output stream to convert it to an image. So this is what I've been working on currently.

****

The period between the start of the second phase and mid of the third phase were really fruitful. I had challenges to solve and that's when I got adrenaline rushes many times.

All in all, the project was very exciting, as opposed to my initial impression, where I thought it to be a web development project and using HTML/CSS and all. I really learned, not only about Octave and its codebase but also many more things like, bitbucket, libcurl, MediaWiki, etc. to name a few. I also learned to manage things on time. This was one of the greatest perks of doing the GSoC.

Needless to say, this all happened with the help of my supportive and understanding mentor, Kai, who timely synced up with me, even though I used to hear from my peers that their mentor doesn't respond timely, etc. He always appreciated my efforts and had a sound idea of what task would take what duration. He also expressed his unsatisfaction at some time, which really gave me a boost to perform better. And not to forget the little things that he noticed, like checking out which student has been putting regular posts on planet octave, etc. I'll always be thankful to him for selecting me for GSoC. Also, I'm thankful to jwe, for introducing such a cool software, that too at no cost. The list may go on without a stop, but my heatiest thanks to all the maintainers and mentors who helped me bring the best out of myself. I'll want to keep working with Octave community in future as well!

To a great "codeful" summer, 2018!

****

[Link](https://bitbucket.org/me_ydv_5/octave/commits/branch/ocs) to BitBucket repo.
