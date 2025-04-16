---
title: Fighting Game Input Buffer
date: 2025-04-10 12:00:00 -500
categories: [project,solo]
tags: [cpp, csharp, unity, unreal]     # TAG names should always be lowercase
---

# Fighting Game prototype

I love fighting games. It's my favourite gaming genre & always try to play the latest of fighting games. Over the years studying & working in game development, I've spend time researching how fighting games are developed with the goal of (hopefully) develop a fighting game in the future.

Because of this, I've been prototyping various systems to not only get a working fighting game combat system, but also one that feels satisfying for the player. I've created a protorype on both Unity & UE5 that functionally works the same, but in this post I'm going to focus & explain the UE5 implementation.

# The Input buffering system

<center>
<img src="../assets/images/FightingGame/InputBufferExample.gif" width="500" alt="hello!"/>
</center>

One of the systems that took the most time figuring out was a input buffering system. Without a system where the input system would be linient to the timing of the player's input, the player would have to be frame-perfect time the want to combo their move (As someone who tried to perform 1-frame combo's in Ultra Street Fighter 4, a game that runs at 60 FPS, it isn't easy at all). A input buffering system provides the player a window of opportunity to press their button at the "right" timing. The "right" timing could be anything, from 3 frame window to a 20 frame window.

I start with creating the input buffer class. Inside the input buffer class contains an array of input buffer items & the buffer window. On initialization, the input buffer initializes an Input Buffer Item based on the buttons that are set & used for the game.

```cpp
class BEATEMUP_API InputBuffer
{
public:

	static const int BufferWindow = 12;

	InputBuffer();
	InputBuffer(TArray<MotionInput*>& inputs);
	~InputBuffer();

	void Initialize();
	void BufferUpdate();

	TArray<InputBufferItem*> InputBufferItems;
	TArray<MotionInput*> MotionInputs;
};
```

An input buffer item is a class that functionally works as a button for the system. It contains an array of input state items.

```cpp
class BEATEMUP_API InputBufferItem
{
public:

	InputBufferItem();
	InputBufferItem(EInputType button);
	~InputBufferItem() {};

	void InputCheck();
	void SetHoldUsed(int index, int time, bool used, bool motion);
	void SetInputActionPressed(bool pressed);

	bool InputActionPressed = 0;

	EInputType InputType;

	TArray<InputStateItem> Buffer;
};
```

An input state item is an object that checks if it has been used within the input buffer. It contains if the current buffer is still being pressed or is released, how long it has been pressed or if the state item has been used. The IsUsed it to check wether the buffer has already been used for something, such as transitioning states, preventing it for being used multiple times.

```cpp
class BEATEMUP_API InputStateItem
{
public:

	bool CanExecute();
	bool CanMotionExecute();

	void HoldUp();
	void ReleasedUp();
	void SetHoldUsed(int time, bool used, bool motion);
	void SetUsed(bool used);
	void SetMotion(bool used);

	int HoldTime = 0;

	bool IsUsed = 0;
	bool MotionUsed = 0;
};
```

When a input buffer item initializes, it creates a input state item based on the input buffer window.

```cpp
UInputBufferItem::UInputBufferItem()
{
    for (int i = 0; i < UInputBuffer::m_BufferWindow; i++)
    {
        m_Buffer.Add(UInputStateItem());
    }
}
```

To update the input buffer system, you must call the BufferUpdate method in a tick function. The input byffer system loops through all the registered buttons & calls a function called InputCheck, which updates the first index of the button's buffer based if the button is being held or being released. Then, we loop through the other indexes of the button's buffer & update the data of the buffers based on the values of it's previous buffer index.

```CPP
void InputBuffer::BufferUpdate()
{
    if (InputBufferItems.Num() > 0)
    {
        for (auto bufferItem : InputBufferItems)
        {
            // Checks if the button is pressed & update the first buffer item
            bufferItem->InputCheck();

            if (bufferItem->Buffer.Num() > 1)
            {
				// Read the buffer from the top to the bottom
                for (int i = bufferItem->Buffer.Num() - 1; i > 0; i--)
                {
                    // Moves the buffer data lower
                    bufferItem->SetHoldUsed(i, bufferItem->Buffer[i - 1].HoldTime, bufferItem->Buffer[i - 1].GetUsed(), bufferItem->Buffer[i - 1].MotionUsed);
                }
            }
        }
    }
}
```

The input check updates the first index of the buffer based on if the button is pressed or not.

```CPP
void InputBufferItem::InputCheck()
{
    if (Buffer.Num() > 0)
    {
		// If the button is pressed/held, call HoldUp, otherwise call ReleaseUp
        if (InputActionPressed)
        {
            Buffer[0].HoldUp();
        }
        else
        {
            Buffer[0].ReleasedUp();
        }
    }
}
```

HoldUp is a function that updates the hold time of the buffer. ReleaseUp is a functions that resets the buffer's data. I only check for the first index of the buffer, since the buffer update shifts the data to the next buffer.

```cpp
void InputStateItem::HoldUp()
{
    if (HoldTime < 0)
    {
        HoldTime = 1;
    }
    else
    {
        HoldTime += 1;

        if (HoldTime > 99)
            HoldTime = 99;
    }
}

void InputStateItem::ReleasedUp()
{
    if (HoldTime > 0)
    {
        HoldTime = -1;

        IsUsed = false;
        MotionUsed = false;
    }
    else
    {
        HoldTime = 0;
    }
}
```

When a button buffer index is registered pressed but not yet used, the button IsUsed state needs to be set. To check if the buffer hold time has just been set & IsUsed hasn't been set yet, I call the CanExecute boolean.

```cpp
bool InputStateItem::CanExecute()
{
    if (HoldTime == 1 && IsUsed == 0)
    {
        return true;
    }

    return false;
}
```

If the CanExecute returns true, the IsUsed must be set to true, otherwise this check continues for the other buffer indexes. To set the buffer IsUsed to true, I call the SetUsedTrue method.

```cpp
void InputStateItem::SetUsedTrue()
{
    IsUsed = true;
}
```

With the input, we want to see if the player has just pressed the button, so we want to check if the any of the buffer hold time values is 1. If the value is 1, we first update the buffer item that it has been used. This prevents that the button is being pressed twice in short succession. In the example below, we switch states based on the input & if the buffer has already been used or not.

```CPP
bool ABaseFighter::InputCheck(EInputType input)
{
	for (int i = 0; i < ReturnInputBuffer()->InputBufferItems.Num(); i++)
	{
		if (ReturnInputBuffer()->InputBufferItems[i]->Buffer.Num() > 0)
		{
			if (ReturnInputBuffer()->InputBufferItems[i]->InputType == input)
			{
				for (int j = 0; j < ReturnInputBuffer()->InputBufferItems[i]->Buffer.Num(); j++)
				{
					// Check if the buffer has the requirements to set used to true (Value is 1 & IsUsed is false)
					if (ReturnInputBuffer()->InputBufferItems[i]->Buffer[j].CanExecute())
					{
						ReturnInputBuffer()->InputBufferItems[i]->Buffer[j].SetUsedTrue();

						return 1;
					}
				}
			}
			else
				continue;
		}
	}

	return 0;
}

UBaseState* ABaseFighter::ReturnAttackState()
{
	// Transition to the state if InputCheck returns true
	if (InputCheck(EInputType::LightPunch))
		return DuplicateObject(LightAttack.GetDefaultObject(), nullptr);
	if (InputCheck(EInputType::MediumPunch))
		return DuplicateObject(MediumAttack.GetDefaultObject(), nullptr);
	if (InputCheck(EInputType::HeavyPunch))
		return NewObject<UJumpState>();

	return nullptr;
}

UFightState* UGroundedComboAttackState::HandleInput(ABaseFighter& fighter)
{
	if (CurrentFrame >= MinCancelFrame && CurrentFrame <= MaxCancelFrame)
	{
		if (fighter.HasHitEnemy())
		{
			if (fighter.InputCheck(State.Input))
				return DuplicateObject(m_State.State.GetDefaultObject(), nullptr);
		}
	}
	
	return UGroundedAttackState::HandleInput(fighter);
}

```

Before the input buffer can be used, the buttons need to added to the buffer system. Since this project is being made in UE5, I'm using the enhanced input system to add the buttons to the buffer system & detect when the button are being pressed/released.

```cpp
// Called to bind functionality to input
void ABaseFighter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	UEnhancedInputComponent* EnhancedInput = Cast<UEnhancedInputComponent>(PlayerInputComponent);

	for (int i = 0; i < MappingContext->GetMappings().Num(); i++)
	{
        // Create a new inut buffer item
		UInputBufferItem* item = new UInputBufferItem();

		EInputType input = InputFromString(MappingContext->GetMappings()[i].Action.GetName());

        // Check if a button of the same type already exists or not
		if(input != EInputType::None)
		{
            // Set the input type
			item->AssignInputType(input);

            // Add the input item to the buffer
			BufferHandler->InputBufferItems.Add(item);

            // Bind the action pressed & released events to the enhanced input
			EnhancedInput->BindAction(MappingContext->GetMappings()[i].Action, ETriggerEvent::Triggered, this, &ABaseFighter::ActionPressed, BufferHandler->InputBufferItems.Num() - 1);
			EnhancedInput->BindAction(MappingContext->GetMappings()[i].Action, ETriggerEvent::Completed, this, &ABaseFighter::ActionReleased, BufferHandler->InputBufferItems.Num() - 1);
		}
	}
}
```

With all of this setup, we can now perform combo's with a lineancy to not be precise with pressing inputs to perform combos or special moves that require a special input! The gif below showcases me performing a combo & performing a dash by pressing Forward->Forward (pressing right twice in quick succession) after finishing the combo.

<center>
<img src="../assets/images/FightingGame/FightingGameInputExample.gif" width="500" alt="hello!"/>
</center>
Note: The red boxes are collision boxes for attacks, such as kicks & grabs.

# Closing Thoughts

Personally, I'm very happy with this implemention. It does require a lot of setup to make it work, but from older iterations that I've created, for my fighting game project it works as desired & I've managed to extend it to also check for special inputs, which I may showcase in the future.

I'll be explaining other systems that I've created for the fighting game, such as how I'm collision handling using my Hitbox component & how I've setup my state machine for the player character. Until then, feel free to checkout my code on my [github!](https://github.com/AyoubLam1997/ProjectFightClub)
