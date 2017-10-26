---
layout: post
title: Windows prevent folder deletion
tags: [windows, security, acl]
---

Suppose we have following folders structure:

```
C:\Media
├───Music
└───Video
```

And we want:

 * Disallow any modifications on Media folder
 * Allow any modifications in Media subfolders

First of all we need to Disable inheritance on Media folder

![Disabling inheritance](http://mac-blog.org.ua/images/win-acl/1.png)

While disabling inheritance you will be asked what you like to do with the current inherited permissions, choose **Convert inherited permissions into explicit permissions on this object**

![Disabling inheritance](http://mac-blog.org.ua/images/win-acl/2.png)

Now remove Users Special permissions (they allowing Users group create files and folders)

![Users special permissions](http://mac-blog.org.ua/images/win-acl/3.png)

At this moment users group will be able to access all items but will not be allowed to modify anything

Now you need **Disable Inheritance** on Music and Video folders as described above.

And add new special permissions for Music and Video folders:

![Special permissions](http://mac-blog.org.ua/images/win-acl/4.png)

At this moment users are not allowed to modify Media folder but allowed to modify Music and Video folders content.

If for some reason you want also to make the same for your own user you should repeat all steps above but for Administrators group, also you should remove CREATOR_OWNER permissions.