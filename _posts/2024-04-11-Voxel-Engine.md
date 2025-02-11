---
title: Voxel Engine
date: 2024-04-11 12:00:00 -500
categories: [project,solo]
tags: [engine,cpp]     # TAG names should always be lowercase
---

<center>
<img src="../assets/images/Voxel/ps5_final_showcase.gif" alt="Playing our voxel engine tech-demo game on the PS5"/>
</center>

In my second year at **Breda University of Applied Sciences** (A.K.A. **BUAS**), I worked on a team project where we had to create a [custom voxel engine](https://github.com/BredaUniversityGames/2324-Y2C-PR-Voxel). The engine should be able to run a game on both PC & the Playstation 5. Using the engine, we created a game where the player can walk around in a random-generated voxel world, removing voxels which contains resources that the player can collect. The player can also place voxels to platform on.

I worked on the following:
- Physics
- Collision detection & handling
- Player Controller

<H3> Physics </H3>

For the physics, there is the expected gravity & rigidbody. The rigidbody (which was dubbed Body in our project) is a physics component which enables actors in the game world to be affected by physics, such as gravity or when a rigidbody wants to push another rigidbody. 

One thing that I added to the rigidbody is Drag. Based on the mass & drag of the rigidbody, it gets wind resistance on the opposite direction of where the player is moving, slowing down the rigidbody

```cpp
class Body
{
//Code...
inline void Update(float dt)
    {
        if (m_type == Dynamic)
        {
            m_linearVelocity += (m_force * m_invMass) * dt;
        }
        // This implementation mimics Unity's version of drag.

        m_linearVelocity = m_linearVelocity * (1.f - dt * m_drag);

        m_position += m_linearVelocity * dt;
    }
}
//Code...
```

This also means that when the player is falling from a high platform, there is a max cap of how fast the player falls. This prevents the rigidbody constantly adding velocity without a limit. 

<center>
<img src="../assets/images/Voxel/DragShowcase.gif" alt="Playing our voxel engine tech-demo game on the PS5"/>
</center>
This implementation is based on Unity's Drag implementation.

<H3> Collision Handling </H3>

Since voxels are just boxes, I check for Axis-Alligned-Bounding-Boxes (AABB) collision. To check for AABB collision, I use the Minkowski Difference algoritm.

```cpp
bool CollisionCheckAABBMinkowski(const AABB& minkowski)
{
    if (minkowski.m_minAABB.x <= 0 && minkowski.m_minAABB.y <= 0 && minkowski.m_minAABB.z <= 0 && minkowski.m_maxAABB.x >= 0 &&
        minkowski.m_maxAABB.y >= 0 && minkowski.m_maxAABB.z >= 0)
    {
        return true;
    }

    return false;
}

// Resolve AABB collision
void World::ResolveCollision(CollisionData& collision, Body& body1, Body& body2, AABB& minkowski)
{
    // if both bodies are not dynamic, there's nothing left to do
    if (body1.GetType() != Body::Type::Dynamic && body2.GetType() != Body::Type::Dynamic) return;

    vec3 penetration = CalculatePenetration(collision, minkowski);

    if (body1.GetType() == Body::Type::Dynamic) body1.m_position += (penetration / 2.f);
    if (body2.GetType() == Body::Type::Dynamic) body2.m_position -= (penetration / 2.f);
}

void World::UpdateCollisionDetection()
{
    CollisionData collision3D;

    const auto& view_aabb = Engine.ECS().Registry.view<Body, AABB>();

    for (const auto& [entity1, body1, box1] : view_aabb.each())
    {
        if (body1.GetType() == Body::Type::Static) continue;

        for (const auto& [entity2, body2, box2] : view_aabb.each())
        {
            if (entity1 == entity2) continue;

            if (body2.GetType() != Body::Type::Static && entity1 > entity2) continue;

            AABB minkowski(vec3((body1.GetPosition() + box1.m_minAABB) - (body2.GetPosition() + box2.m_maxAABB)),
                           vec3((body1.GetPosition() + box1.m_maxAABB) - (body2.GetPosition() + box2.m_minAABB)));

            CollisionData collisionData;

            if (CollisionCheckAABBMinkowski(minkowski))
            {
                if (!box1.m_isTrigger && !box2.m_isTrigger)
                {
                    ResolveCollision(collisionData, body1, body2, minkowski);
                }

                RegisterCollision(collisionData, entity1, entity2);

            }
        }

        //Code...
    }
}
```

On the gif below, you see the player box collider wireframe highlighted in green walking to various voxels, preventing the colliders from clipping from each other.

<center>
<img src="../assets/images/Voxel/CollisionTest.gif" alt="Playing our voxel engine tech-demo game on the PS5"/>
</center>

<H3> Player Controller & Movement </H3>

Since the player needs the ability to be controlled either by mouse & keyboard if the game is being run on PC or gamepad when the game is being run on PS5, the controller needed some logic based on the control scheme.

I created a controller class called FPSController. On the FPS controller update function, we check for the control type & call the relevant controls function in the form of a control type enum. Below is a code snippet of the update method & the gamepad control method.

```cpp
void FPSController::Update(float dt, Transform& transform)
{
    UpdateDirection();
    UpdateCameraDirections();

    float speed{};

    switch (m_movementType)
    {
    case CameraMovementType::Fly:

        speed = m_flyMovementSpeed * dt;

        break;

    case CameraMovementType::Grounded:

        speed = m_walkMovementSpeed * dt;

        break;
    }

    switch (m_controlType)
    {
    case ControlType::MouseKeyboard:

        MouseKeyboardControl(speed, transform.Translation);

        break;

    case ControlType::Gamepad:

        GamepadControl(speed, transform.Translation);

        break;
    };

    transform.Translation.y += m_cameraHeightOffset;

    const auto viewInv = inverse(View(transform.Translation));
    Decompose(viewInv, transform.Translation, transform.Scale, transform.Rotation);
}

void FPSController::GamepadControl(const float velocity, vec3& transform, Body& body)
{
    switch (m_movementType)
    {
    case CameraMovementType::Fly:

        body.SetLinearVelocity(
            (m_front * velocity) * -Engine.Input().GetGamepadAxis(0, Input::GamepadAxis::StickLeftY));
        break;

    case CameraMovementType::Grounded:

        // TODO: Controller ID of the player needs to be saved
        
        if (Engine.Input().GetGamepadButtonOnce(0, Input::GamepadButton::South) && m_isGrounded)
        {
            body.AddForce(vec3(0, m_jumpForce, 0));
        }

        vec3 inputY = vec3(cos(radians(m_yaw)), 0, sin(radians(m_yaw))) *
            -Engine.Input().GetGamepadAxis(0, Input::GamepadAxis::StickLeftY) * velocity;
        vec3 inputX =
            vec3(m_right.x, 0, m_right.z) * Engine.Input().GetGamepadAxis(0, Input::GamepadAxis::StickLeftX) * velocity;

        body.AddForce((inputY + inputX));
        break;
    }
}
```

When the game initializes, it checks on what platform the game is. Based on the platform, we initialize the player with the corresponding control scheme (i.e. if the platform is the PS5, initialize the player with the gamepad controls)

```cpp
GameSystem::GameSystem()
{
    const auto cameras = Engine.ECS().Registry.view<Camera>();
    if (cameras.empty())
    {
        // Initializing player entity
        const auto entity = Engine.ECS().CreateEntity();

        auto& transform = Engine.ECS().CreateComponent<Transform>(entity, glm::vec3(-0.144f, -0.074f, -0.043f), glm::vec3(1.0f), glm::vec3(0.0f));

        const float aspectRatio = static_cast<float>(Engine.Device().GetWidth()) / static_cast<float>(Engine.Device().GetHeight());

        auto projection = perspective(radians(90.0f), aspectRatio, 0.001f, 500.0f);

        auto& camera = Engine.ECS().CreateComponent<Camera>(entity);
        camera.Projection = projection;

#ifdef BEE_PLATFORM_PC
        // Create FPS controller & set control mode to Mouse & keyboard state
        auto& fps = Engine.ECS().CreateComponent<FPSController>(entity, ControlType::MouseKeyboard, CameraMovementType::Grounded, 30.f, 1.f, 15.f);
        // Initializing other required components...

// Prospero is the PS5 platform
#elif BEE_PLATFORM_PROSPERO
        // Create FPS controller & set control mode to Gamepad state
        auto& fps = Engine.ECS().CreateComponent<FPSController>(entity, ControlType::Gamepad, CameraMovementType::Grounded, 30.f, 1.f, 15.f);
        // Initializing other required components...
    }
}
```

<H3> Lessons learned & closing thoughts </H3>

This project gave me as a gameplay developer not only the chance to implement physics, collision detection & handling & controls logic in a custom engine, but it gave me also a deeper insight on how Engine & Graphic developers perceive a project like this & how I should work with them to create the best possible product that we can develop. Adding on top that we had to make the engine work on PS5 made this project decently challenging.

Still, the project was incredibly insightful & I got the chance to work amazing people that I'm proud to call my peers.
