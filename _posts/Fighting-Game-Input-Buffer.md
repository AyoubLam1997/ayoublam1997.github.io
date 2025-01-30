---
title: Fighting Game Input Buffer
date: 2024-07-07 12:00:00 -500
categories: [project,solo]
tags: [cpp, unity, unreal]     # TAG names should always be lowercase
---

# Fighting Game prototype

I love fighting games. It's my favourite gaming genre & always try to play the latest of fighting games.

Because of this, I've developed some prototypes to try & create my own fighting game system.

For a fighting game, I need the following systems:
- A state machine.
- An input buffering system.
- A customizable collision box, which I'll be calling a hitbox from this point forward.

# Hitbox component

The goals for the hitbox component are the following:
- The hitbox should only check for collision when assigned to do so.
- The collision resolution should be customizable.

The hitbox has 3 states: Open, Colliding & Closed. Open means that the hitbox is checking for collision but hasn't collided with anything yet, collided means it is checking & is colliding with something & closed means it isn't checking for collision at all.

The hitbox also has a responder attached to it. The responder contains a method that's being called when an object has collided with the hitbox for the first time. The responder is customizable, so you can design what should happen when first-time collision is detected with an object.

SHOW CODE & EXAMPLES OF THE RESPONDER

# State machine

Each state has their own contained logic, preventing it from influencing either the system or other states.

Each state is derived from BaseState

# Input buffering system

The input buffering system gives the player lineancy to press a button or to press a specific button combination so that the player performs their desired action.



In games where combat is involved, it is desired that the player has a window where they can 

To create a working input buffering system, I need to fulfill the following criteria:

For each button, a input buffer item is created. Each input buffer item contains an array buffer state item with the size of how big the buffer should be. In this case, the buffer window is 12 frames.

Another thing we have to do is updating the buffer that the button has been pressed. If we don't, the system would otherwise handle it as if the button has been pressed again, even though the button is still being held. We do this with a simple boolean that we set to see if the buffer has been used or not.

CODE SNIPPET
