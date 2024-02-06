---
title: Making Sense of Catalina's Notarization Requirements
date: 2019-07-24
layout: post
author: andrew
image: assets/images/notarize.jpg
categories: [ mac, "system administration" ]
---

A few weeks ago, Apple announced MacOS 10.15 - Catalina.  Along with this they also introduced a requirement to sign and notarize all `.pkg`, `.dmg`, kexts, and Applications.

> Mac apps, installer packages, and kernel extensions that are signed with Developer ID must also be notarized by Apple in order to run on macOS Catalina. This will help give users more confidence that the software they download and run, no matter where they get it from, is not malware by showing a more streamlined Gatekeeper interface.

[Source](https://developer.apple.com/news/?id=06032019i)

So what does this mean for us Mac SysAdmins? It means we're going to have to do a lot of work pushing companies to get their shit together.

# Checking existing software for compatability

We can easily determine if a pkg will run on catalina using existing tools. Be sure to have XCode installed before running these checks

## PKGs

```
$ pkgutil --check-signature <path to pkg>

$ pkgutil --check-signature 1Password-7.3.1.pkg

Package "1Password-7.3.1.pkg":
  Status: signed by a certificate trusted by Mac OS X
  Certificate Chain:
    1. Developer ID Installer: AgileBits Inc. (2BUA8C4S2C)
      SHA1 fingerprint: 3F F5 AB E5 D3 F8 3D FD 81 57 C8 6A 30 19 C7 73 99 BA 0D 25
      -----------------------------------------------------------------------------
    2. Developer ID Certification Authority
      SHA1 fingerprint: 3B 16 6C 3B 7D C4 B7 51 C9 FE 2A FA B9 13 56 41 E3 88 E1 86
      -----------------------------------------------------------------------------
    3. Apple Root CA
      SHA1 fingerprint: 61 1E 5B 66 2C 59 3A 08 FF 58 D1 4A E2 24 52 D1 98 DF 6C 60

```

This package has been signed by AgileBits Inc. Because it's been signed, it must also now be notarized to work on MacOS Catalina. We can check the notarization of an app with

```
$ stapler validate <path to pkg>

$ stapler validate 1Password-7.3.1.pkg

Processing: /Users/aduss/Downloads/1Password-7.3.1.pkg
The validate action worked!
```

It works! This pkg has been signed and notarized. It will work on Catalina:

![1PasswordWorkingOnCatalina](https://i.imgur.com/a2BpwdE.png)

## DMGs

DMGs can also be checked to see if they will work on Catalina.  The same process applies, but with slightly different commands. We first need to check that the DMG is signed, and then check that it's been notarized.

```
$ spctl -a -t open --context context:primary-signature -v <path to dmg>

$ spctl -a -t open --context context:primary-signature -v Dropbox-76.4.126.dmg

./Dropbox-76.4.126.dmg: accepted
source=Developer ID
```

This Dropbox dmg has been signed. Let's see if it's notarized. We can use the `stapler validate` command again to check.

```
$ stapler validate <path to dmg> 

$ stapler validate Dropbox-76.4.126.dmg

Processing: Dropbox-76.4.126.dmg
Dropbox-76.4.126.dmg does not have a ticket stapled to it.
```

No ticket = Not Notarized. Dropbox 76.4.126 will not work on Catalina

![DropboxDoesNotWorkOnCatalina](https://i.imgur.com/M6I81qE.png)

## Applications

In general, Applications must be signed before they can be installed onto your mac. But if you're using an old version of MacOS or just want to verify, you can use the `codesign` command in your terminal

```
$ codesign -dv <Path to App>

$ codesign -dv /Applications/Spotify.app/

Executable=/Applications/Spotify.app/Contents/MacOS/Spotify
  [ .. snip .. ]
Signature size=9046
  [ .. snip .. ]
```

This Spotify app is signed. Let's see if it's notarized

```
$ stapler validate <Path to App>

$ stapler validate /Applications/Spotify.app

Processing: /Applications/Spotify.app
Spotify.app does not have a ticket stapled to it.
```

Womp Womp =(  Spotify is not notarized. It does not work on Catalina.

![SpotifyScreenshot](https://i.imgur.com/nbJpX0z.png)

# So how can we fix this?

Unfortunately, there is little we can do as consumers of applications. If we don't own the code, we can't notarize it.  Reach out to the support teams of the software you're trying to install and tell them their product doesn't work on MacOS 10.15 Catalina.

If you do own the code, you'll need to notarize your application. Apple has created a great guide on how notarize your application so it can work on 10.15. Feel free to [read this article](https://developer.apple.com/documentation/security/notarizing_your_app_before_distribution) to get started.

