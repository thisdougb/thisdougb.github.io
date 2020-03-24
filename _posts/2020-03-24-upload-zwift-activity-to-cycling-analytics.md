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

To simplify things, I created a [video](https://vimeo.com/400290190) to walk you through it.

But for the hasty, this is the gist of it:
### Do These Two Steps Once
###### Generate the bookmarklet with the new code
![bookmarklets.org](/images/bookmarklet-maker.png)

###### Add the bookmarklet to your browser toolbar (replace TOKEN!)
![bookmarklets.org](/images/add-bookmarklet.png)

### Do These Steps To Upload Each Activity
###### Go to the settings of an activity you want to upload
![bookmarklets.org](/images/zwift-activity.png)

###### Click the bookmarklet you just created, to insert the CA button
![bookmarklets.org](/images/click-bookmarklet.png)

###### Click the SEND TO CA button, it ticks when done
![bookmarklets.org](/images/click-upload.png)

###### View your ride in Cycling Analytics
![bookmarklets.org](/images/activity-uploaded.png)

For any questions/suggestions I'm on Twitter [@thisdougb](https://twitter.com/thisdougb)
