---
title: "Increasing Playback Speed on Kodi LibreELEC on Raspberry Pi"
layout: post
date: 2018-02-02
author: andrew
categories: [ "raspberry pi" ]
---

Increasing the playback speed of a video has never been easier with Kodi on your Raspberry Pi.  To enable this, we must do two things. First, we need to enable playback syncing.

This can be done by enabling the setting under:

General Settings > Player > Videos > Sync playback to display

Then we have to create a new keymap. Create a new file, `keyboard.xml` and add the following content:

```
<?xml version="1.0" encoding="UTF-8"?>
<keymap>
  <FullscreenVideo>
    <keyboard>
      <opensquarebracket>PlayerControl(TempoDown)</opensquarebracket>
      <closesquarebracket>PlayerControl(TempoUp)</closesquarebracket>
    </keyboard>
  </FullscreenVideo>
  <VideoMenu>
    <keyboard>
      <opensquarebracket>PlayerControl(TempoDown)</opensquarebracket>
      <closesquarebracket>PlayerControl(TempoUp)</closesquarebracket>
    </keyboard>
  </VideoMenu>
</keymap>
```

Now, just upload `keyboard.xml` to your Raspberry Pi under the `/storage/.kodi/userdata/keymaps` directory. I find using SSH or SFTP easiest.

Lastly, just restart your Pi and you should now be able to use the `[` key on your keyboard to decrease the playback speed, and `]` to increase the playback speed!
