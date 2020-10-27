---
layout: post
title:  "Zero to hero with OBS Studio on Fedora 32"
date:   2020-10-27 10:30 +0200
author: hadez
categories: howto
---

[OBS](https://obsproject.com/) is amazing, if it just were to work out of the box the way I want.
The problem is mostly that I'm trying to use it on Linux, because on Windows you can just use [Streamlabs OBS](https://streamlabs.com/streamlabs-obs) and be done with it.

## What I want

- Virtualcam support to stream OBS output to arbitrary v4l2-accepting software
- Greenscreen to allow for all the cool shenannigangs
- Make it nice
    - Basic audio filtering to get rid of noise
    - Even greenscreen lighting so it actually works properly

Installing OBS is simple enough, but then the interesting bits start to happen.

## The Problems

- Streamlabs is not available on Linux, so there's no easy, out-of-the-box solution on Linux that offers the above
- Original obs-v4l2sink has not been maintained for 2 years and does no longer compile using the original instructions
- Building OBS from scratch also takes more work than just running cmake && make. For some reason it does not seem to properly check for required dependencies and even a successful cmake run does not imply successful compilation.
- And if that were not enough, after updating to Fedora >=31 good ol' Wayland seems to cause the video preview panes in OBS to be rendered fully transparent

## The Solution

### Transparent OBS

The simple things first: you'll need to [set an ENV variable](https://obsproject.com/forum/threads/fedora-31-preview-window-is-transparent.112877/):

```
$ cat obs.sh 
#!/bin/sh
QT_QPA_PLATFORM=xcb obs
```

### Virtual Camera

You'll need two things:

- v4l2loopback, a kernel module that enables you to create virtual v4l2 "webcam" devices. This allows you to have one software stream to said device while another reads from it.
- obs-v4l2sink, an OBS plugin that allows you to stream to a v4l2loopback device.

After failing to build the latter from [CatxFish's](https://github.com/CatxFish/obs-v4l2sink) unmaintained repositiry, I stumbled over [seii's](https://github.com/seii/fedora-green-screen/blob/master/README.md) writeup that ultimately lead me to [blues-man's copr](https://github.com/blues-man/obs-v4l2sink-plugin-fedora).
Follow the steps outlined by [seii](https://github.com/seii/fedora-green-screen/blob/master/README.md) and/or blues-man's README to both install v4l2loopback as well as obs-v4l2sink.

You probably want to persist any parameters you pass to modprobe using a new file in `/etc/modprobe.d`.
I for one simply use the one's given by blues-man (more or less):

```
$ cat /etc/modprobe.d/v4l2loopback.conf         
options v4l2loopback video_nr=42 card_label="obs" exclusive_caps=1
```

Creating a virtual webcam is now as easy as opening OBS, massaging your scene into place (we'll get to that), and choosing `Tools > V4l2sink; Device=/dev/video42; [Start]`.

### Blinded by the lights

The easiest and cheapest way to improve your streaming game is to have some proper lights in your face and no lit windows behind you.

Want soft and bright light? The magic word on Aliexpress et al is ["COB LED"](https://de.aliexpress.com/wholesale?catId=0origin=y&SearchText=cob+led).
I'm using 220x120cm 12V 70W modules.

Do yourself a favor and get one with a power supply or even better get multiple COBs and a single power supply able to drive them together (in parallel).
You can also use an old wall-wart/powerbrick you have still around that's 12V and sufficient amperage here (I did).

For my face lighting I use two 70W COBs diffused through sheets of white printer paper.

### Video color correction

If you're like me and picked warm-white COBs in the previous step, you might look a bit sun-burned in the video ;)

This is an easy enough fix using a LUT filter in OBS:

- Take a screenshot of the OBS window or webcam video showing yourself in the position you want to use later on with the light setup you're planning on using. If you have a [gray-card](https://www.amazon.de/-/en/gp/product/B00KCPBLWO/), now's the time to hold it next to your face, alternatively wear something that has neutral gray color in it or hold up a white sheet of paper.
- Open the screenshot in Gimp
- Select `Colors > Levels`
- Use the gray mid-tone color dropper to pick your gray card
- Try using the white/black points droppers on white/black reference surfaces and see if you like the result
- If it looks good, save these settings as a preset right there in the `Levels` dialogue window
- Open `/usr/share/obs/obs-plugins/obs-filters/LUTs/original.png` in Gimp
- Apply the preset you just saved to it
- Save the result somewhere under a suitable name (my_obs_lut.png or something is fine)
- Go to OBS and right-click your webcam source and select `Filters`
- Add an `Apply LUT` filter
- Select your previously saved modified LUT
- Observe the magic!

### Greenscreen tuning

First of all, just get a cheap greenscreen kit including the stand to prop it up on Amazon or Ebay.
It should not set you back more than 30€ and it's a worthwhile investment.
If it's one of the Tyvek sheet based screens, I recommend the ones that come with multiple colors.
You can use the white backdrop as a diffusor for what's to come next!

Remember the COB LEDs from before?
Get two more.

OBS's keying (chroma, color, luma) works best if the background is evenly lit and uniform in color.

I simply backlit my screen using two COBs to both get the color and lighting on the screen as even as possible and bright enough as to not be too close to black to allow clean keying.

Now follow these steps:

- Backlight the greenscreen untill it looks somewhat even
- Take a screenshot of the OBS window or just the webcam view showing ideally nothing but the greenscreen background
- Use gimp and select the greenscreen area
- Heavily apply Gaussian blur
- Use the color picker to select an average-ish green color
- Back in OBS, right-click your camera source and select `Filters`
- Add a chroma key (not color or luma)
- Select the custom color option from the drop-down and set it to the one you color-picked in gimp
- Adjust the sliders for similarity (try to keep it as low as possible, below 100 for me works) and smoothness (keep it very low, 20 or even single digit amouts work for me). The key here is: the more even your greenscreen is lit, the lower you can set the parameter

Tip: If you still have a gradient and the chroma key cannot fully key out all of the background, add multiple choma key instances set to slightly different colors (all picked via Gimp).


### Audio tuning

Get a half-way decent microphone.
I use the a [Fifine USB microphone](https://www.amazon.de/-/en/gp/product/B07QC5W7G9/) and a [flex-arm](https://www.amazon.de/-/en/gp/product/B073VJKD9Q) to hold it up.
It does not have the best noise floor and is somewhat sensitive to environment noise, but we'll fix that soon enough.

- Open up OBS
- Use headphones! Otherwise the next step might produce nasty feedback!
- Go to `Edit > Advanced Audio Properties`
- Then, for the entry of your microphone, pick `Monitor and Output` from the `Audio Monitoring` column
- Close the properties window

You should now hear yourself speak on your headphones.

- Right-click your audio source (the microphone input) and select `Filters`
- Add a `Noise Suppression` filter. You should immediately hear the noise floor drop away entirely
- Add a `Noise Gate` filter and adjust the open/close threshold relatively high. Just high enough so the microphone only triggers when you speak, but not when you type or set down your tea or coffee mug on the table :)
- Re-order the filters so the gate is above the noise supression filter
- Add a `Compressor` filter. This will make your voice input more even and softly suppress strong volume changes. Default settings should suffice for starters.
- Re-order the filters so the compressor is below the other two

Congrats, you now have a half-decent audio setup!

The only thing missing is getting that audio into your target application as well.
That's an ongoing process and possibly addressable by pointers given [here on Github](https://github.com/CatxFish/obs-v4l2sink/issues/34).

## Coming up

There's still a lot of work ahead that I want to tackle:

- Remote controlling OBS from mobile phone
- OBS browser overlay
- DIY Teleprompter
- Nikon D810 as webcam
- Improve the greenscreen setup further
- Get audio from OBS into consumer application

But that's for another time :)

