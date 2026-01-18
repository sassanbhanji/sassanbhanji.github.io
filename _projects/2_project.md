---
layout: page
title: CamHacks 25 Project
description: We built an Anrgy Birds game that can be controlled by WhatsApp voice messages
importance: 2
category: work
github: https://github.com/sb-2700/PythonAngryBirds
---

The theme for CamHacks 25, a hackathon hosted by the Computer Science department in Cambridge, was 'Unintended Behaviour'.

Our response to that theme? Using WhatsApp in an unintended way: controlling Angry Birds. We used an open source Pygame implementation of Angry Birds and modified it so that you select your bird and fire it solely using bird noises that you send via WhatsApp.

Although it was a silly project, it involved some interesting engineering. We used Twilio to send the audio files from WhatsApp to our server. To analyse and classify the 'bird noises' we used a pretrained 14 layer CNN with a linear head at the end that we trained ourselves on synthetic data, achieving >90% accuracy. This is pretty impressive given that the audios were human imitations of angry bird noises which made for a challenging classification task on minimal data and compute. We used spectrograms to calibrate the bird noises so that frequency determines shot angle and volume determines shot power.

Thanks to Aaron MacWhirter, Matyas Vecsei and William Goacher for their contributions to this project.
