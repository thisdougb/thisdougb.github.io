---
layout: post
title: "Upload Zwift Activities to Cycling Analytics"
permalink: /cycling/2020-03-25-zwift-to-cycling-analytics
tags: [cycling,zwift,indoor,javascript,analytics]
---
Cycling Analytics [published](https://www.cyclinganalytics.com/blog/2018/10/upload-rides-from-zwift) a method for easily uploading Zwift activities. Unfortunately Zwift has evolved, and the method no longer works.

I updated the method, which is a little different but still a lot less hassle than downloading the fit files and then uploading to CA.



What you will need:
* The [original method](https://www.cyclinganalytics.com/blog/2018/10/upload-rides-from-zwift), you should read this for context
* You Cycling Analytics token (see above instructions)
* The [new code](https://gist.github.com/thisdougb/45221618e66300b05a0c82e875c3b264) which does the magic
* Use this [bookmarklet generator](https://bookmarklets.org/maker/)

To simplify things, I created a [screen capture video](https://vimeo.com/400242509).
But for the hasty, this is the gist of it:

* Generate the bookmarklet with the new code
* Go to your Zwift feed: https://zwift.com/feed
* Click on the activity you want to upload
* Click on the Setting/Detail icon (round gear to the right of the ride title)
* Click on that bookmarklet, which should reveal 'SEND TO CA'
* Click on the 'SEND TO CA' button
* View your ride in Cycling Analytics

For any questions/suggestions I'm on Twitter [@thisdougb](https://twitter.com/thisdougb)
