---
author: micdemarco
comments: true
date: 2016-09-22 02:50:15+00:00
layout: post
slug: posting-pictures-from-google-photos-to-facebook-from-a-windows-laptop
title: Posting pictures from Google Photos to Facebook using a Windows Laptop
tags:
- facebook
- google photos
- picasa
- windows
---

As a Google Photos and Facebook user, it has always bothered me how difficult it is to share pictures from one app to the other from a Laptop.

Doing it from a phone is relatively easy.  You open the Google Photos app, select pictures and share them to Facebook.

Doing the same thing from a laptop is a bit more difficult.

Previously I used Picasa, and used to export and upload my pictures using a plugin (it was not the best user experience).  After Picasa was discontinued, I stopped using it and switched to just using Google Photos from the browser, and the Google Photos Uploader to upload new stuff.  However, I still wanted to have a local copy of my pictures.  Also sharing anything to Facebook from the laptop remained a pain, and performing all the edits in the browser was still no as good as a native app, so I needed a better setup.

I decided to write this blog post after reading the answers to [this question](https://productforums.google.com/forum/#!topic/photos/p9aK7m4sMyM) and share my current setup, which I am quite happy with.

## The Steps

The following steps will show you how to set up Google Drive to sync Google Photos to disk and how to setup Windows Photos for managing Google Photos and sharing photos to other Windows Store apps such as the Facebook app.

1) Set up Google Drive to Sync your pictures to Windows

![2016-09-22_11-44-45](/assets/2016-09-22_11-44-45.png)

2) Install Google Drive and either sync everything  or add the Google Photos folder to the list of synced folders.Note: the first time you do this, you will download your entire pictures library to disk

3) Open the Windows Photos store app

![2016-09-22_11-55-21](/assets/2016-09-22_11-55-21.png)

4) Go to settings and add the your Google Photos folder to the sources

![2016-09-22_11-57-17](/assets/2016-09-22_11-57-17.png)

**Update 05/11/16**

In order for Photos app to be able to read the photos, you will need to set permissions to your Google Drive folder as described in [this article](http://superuser.com/questions/485719/windows-7-index-search-does-not-work-in-google-drive-folder).

![2016-11-05_11-42-11](/assets/2016-11-05_11-42-11.png)

You should now see all your photos in the Photos app

You can edit your photos locally using Windows Photos app (or any other app) and these changes will be synced back to **Google Drive**, however they will **not** be synced back to **Google Photos**.

5) Open the Windows Store and install the Facebook app.  Launch, Login etc...

![2016-09-22_12-09-50](/assets/2016-09-22_12-09-50.png)

6) Go back to the Photos app

7) Find the pictures that you want to share.  Select the pictures, click share and choose Facebook from the list of apps

![2016-09-22_12-17-03.png](/assets/2016-09-22_12-17-03.png)

8) The Facebook app takes over and you can post you pictures like you would do from a phone

![2016-09-22_12-21-14.png](/assets/2016-09-22_12-21-141.png)

9) Done!

## Conclusion

You can use a Windows Pictures and Google Drive to access your Google Photos library locally and also share pictures to other Windows store apps such as Facebook in the same way as you would on a Tablet or Phone.







Unfortunately 2 way sync is fully supported, so any changes made locally to the photos will only be backed to Google Drive and not to Google Photos.












I could not find a Google Photos app in the Windows App Store (come on Google...?).  That would make everything a lot easier.   There is an app called "Client for Google Photos Free" but it would only share links, and they also have a paid version which I did not try.












For people coming from Picasa,  I think this is a good alternative to using Google Photos from the browser.
