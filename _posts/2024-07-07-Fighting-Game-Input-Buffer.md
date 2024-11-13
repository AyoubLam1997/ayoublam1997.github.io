---
title: Fighting Game Input Buffer
date: 2024-07-07 12:00:00 -500
categories: [project,solo]
tags: [networking,cpp, unity, unreal]     # TAG names should always be lowercase
---

# Fighting Game Input Buffer

Fighting game input buffer

In games where combat is involved, it is desired that the player has a window where they can 

To create a working input buffering system, I need to fulfill the following criteria:

<CRITEREA>

For each button, a input buffer item is created. Each input buffer item contains an array buffer state item with the size of how big the buffer should be. In this case, the buffer window is 12 frames.

Another thing we have to do is updating the buffer that the button has been pressed. If we don't, the system would otherwise handle it as if the button has been pressed again, even though the button is still being held. We do this with a simple boolean that we set to see if the buffer has been used or not.

CODE SNIPPET
