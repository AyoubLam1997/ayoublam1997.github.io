---
title: Webcam Dancing Game
date: 2024-07-07 12:00:00 -500
categories: [project,solo,school]
tags: [unity,csharp]     # TAG names should always be lowercase
---

# Webcam Dancing Game

For a project at GLU, I've created a dancing game where the player dances in front of a camera.

The game collects all the pixels. In the next frame, it then checks for any differences. If there are any differences between the frames. If there are any differences between frames, they are represented by the color white, otherwise they're black.

To check if the player is dancing and/or moving enough to increase their score, the game checks how many of the pixels have changed (in percentage). If enough pixels have changed frames, the player's score increases.
