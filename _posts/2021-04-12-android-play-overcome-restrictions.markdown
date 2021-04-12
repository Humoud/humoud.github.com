---
layout: post
title:  "Acquiring Android Apps: Overcoming Google Play's Restrictions"
date:   2021-04-12 00:00:00 +0300
categories: android
---

### Background
When attempting to acquire Android applications from the Google Play (mobile client) store in preparation for analysis  you might find yourself in a position where you are not able to find the application you require no matter what search query you use. However, if you turn to the web version of the Google Play store and login with the same G account as the one which was used on the mobile device initially to search for the application you might actually find the application in question, along with the following message *"This app is not available for your device"* and nothing more, no details are provided.

Example of querying Disney plus on the Google Play mobile client:
![app on web version of play store](/assets/disney_play_mobile_search.png)

Example of querying for Disney plus on the Google Play web client using the same G account:
![app on web version of play store](/assets/disney_playstore.png)

Digging just a little bit deeper, I've noticed a few cases of why *"This app is not available for your device"*: 
1. The application is not campatible with the device, it just cannot be installed successfully (I tried). Got an error message of the mobile device lacking certain libraries:
2. Restrictions:
    1. Location (unsure yet)
    2. Device type (positive)


### Overcoming Google Play's Restrictions
Restriction 2.2 mentioned above can be overcomed by using the [Aurora Store](https://aurora-store.en.uptodown.com/android) application. Aurora allows applications to be installed without a G account. Moreover, it allows you to spoof the mobile device by using the Spoof Manager feature.

Menu:

![Aurora Store Menu](/assets/aurora_store_sidebar.png)

Spoof Manager:

![Spoof Manager](/assets/aurora_store_spoof.png)

Aurora store can be used in 2 modes: anonymous or G account login. I managed to overcome restrictions using the anonymous mode, I.e, I did not need to sign in.

I managed to query for, find, and install the Disney+ application successfully using Aurora (for the sake of this example):

![Disney+ installed](/assets/aurora_store_disney.png)

When using the spoof manager, you must sign out and then sign back in. That is regardless of the login mode used.

### Concerns
The obvious is the authenticity of the applications downloaded. The applications downloaded do seem to have the same functionality of the original applications. However, that does not mean that they have not been patched.

I did notice that the Aurora application seems to establishes some sort of a session with the Google Play store and in that session your device information will be spoofed which will affect what applications you will be able to install. But I noticed this by paying close attention to the UI (it could be just UI and nothing is actually happening in the background). I noticed some random/throwaway G account being used to log into the Google Play store, but again, might be just UI.
