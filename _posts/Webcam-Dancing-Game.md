---
title: Webcam Dancing Game
date: 2024-07-07 12:00:00 -500
categories: [project,solo,school]
tags: [unity,csharp]     # TAG names should always be lowercase
---

# Webcam Dancing Game

<img src="../assets/images/WebcamGame/dancinggif.gif" width="500"/>

For a project at GLU (Grafisch Lyceum Utrecht), I've created a dancing game where the player dances in front of a camera.

The game collects all the pixels. In the next frame, it then checks for any differences. If there are any differences between the frames. If there are any differences between frames, they are represented by the color white, otherwise they're black.

To check if the player is dancing and/or moving enough to increase their score, the game checks if any motion was detection was detected in front of the camera. If there is, the player's score increases.

```C#
// Checks for any form of motion in front of the webcam
private void MotionDetection()
{
    Color[] pixels = m_WebcamTexture.GetPixels();
    Color[] output = new Color[pixels.Length];

    if(m_PreviousFrame != null)
    {
        m_TotalMotion = 0;
        m_Index = 0;

        // Loop through all the pixels
        for(int pixelY = 0; pixelY < m_WebcamTexture.Height; pixelY++)
        {
            for(int pixelX = 0; pixelX < m_WebcamTexture.Width; pixelX++)
            {
                float motion = Mathf.Abs(pixels[m_Index].maxColorComponent - m_PreviousFrame[m_Index].maxColorComponent);
                
                // Set the pixel color to white if there is any movement or black if there isn't any movement
                output[m_Index] = (motion > 0.02f) ? Color.white : Color.black;

                if(motion > 0.02f)
                {
                    m_TotalMotion++;
                }

                m_Index++;
            }
        }

        // Apply the webcam texture (i.e. what the webcam camera renders) & apply it to the target texture
        m_TargetTexture.SetPixels(output);
        m_TargetTexture.Apply();

        m_TotalPercent = m_WebcamTexture.width * m_WebcamTexture.height;
    }

    m_PreviousFrame = pixels;
}
```

Below is a video of me playing the game (WARNING: AUDIO MAY BE LOUD. APOLOGIES IN ADVANCE)

[![Webcam dancing game](https://img.youtube.com/vi/F7mUnIBvoO4/0.jpg)](https://www.youtube.com/watch?v=F7mUnIBvoO4 "Webcam dancing game")
