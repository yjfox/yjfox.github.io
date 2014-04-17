---
layout: default
title: android AsyncTask development
label: android, AsyncTask, Bitmap
time: 2014-04-17
brief: some points about android AsyncTask
---

#{{page.title}}
> {{page.brief}}
**************

Here is something you need to pay attention when accessing Internet in your android APP as it will lead to NetworkNotFound exception if not followed what mentioned next:

- All know we have to add Internet permission in Manifest XML  
- Add function which need Internet access into a class extends AsyncTask, so that it can work properly.   
BTW: AsyncTask is an internal android class you can import it easily.

some other tips for myself: .split('$') will not work, modified it to .split('\\$'). It seems that "$" is one of restricted words in JAVA

end.
{{page.time}}
