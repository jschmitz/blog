---
layout: post
title: Play Audio in FreeClimb - Convert mp3 to 8 bit mono U-Law
date: 2020-05-21 05:17 -0500
---

When communicating with your users over the phone some users do love the robot voice that Text to Speech (TTS) provides and that is a great option from a developer point of view. It's easy to control and update by just changing a string and you get a new message. No muss no fuss.

There likely will come a time when the user base expects more than a robot voice and to take it up a notch you'll need begin to provide pre-recorded audio. Pre-recorded audio can be provided for hold music as well as conversational audio. We'll check out how to get files into the right format with FreeClimb next.

FreeClimb supports playback of single-channel 8 kHz 8-bit μ-law WAV files. When creating your generating or procuring your own audio you might run into a need to change the format to meet that requirement. You can start by getting an audio editor like [Audacity](https://www.audacityteam.org/).

These steps should get you the file format you need:
1. Import the audio file to convert in your Audacity
    1. Drag it into the editor
    1. or
    1. File --> Import --> Audio
1. Ensure you have a mono track
    1. Track
    1. Mix
    1. Mix Stereo Down to Mono
1. Resample
    1. Tracks
    1. New Sample Rate --> 8000
1. Export to the supported file format
    1. File
    1. Export as Wav
    1. Other Uncompressed Files
    1. Header --> WAV (Microsoft)
    1. Encoding --> U-Law
    1. Save
    1. Export

Want to see this visually?  Check out [this video](https://www.youtube.com/watch?v=CeiJtLs1mJ0) to see the process in action.

With that, you have a file to host in your application that FreeClimb will play.
