# 🐦 Angry Birds Clone – Built with Box2D 🐦


This is a clone of the Angry Birds game, built using the Box2D 2.3.2 physics engine and developed in C++. The project replicates the core gameplay mechanics and physics behavior of the original game as closely as possible. It was created as part of an independent study project for my graduate program.

The game runs on a custom 2D engine developed by my professor, which provides support for sprite rendering, debug wireframe drawing, and camera functionality. Sound integration is handled using irrKlang. 
The game is designed for a resolution of 1280x720; fullscreen mode is not supported due to input positioning issues. Debug Mode in the game, depicts how the physics bodies really look and how they react to each other.

[YOUTUBE LINK TO DEMO](https://youtu.be/xsb5FvlC20Y?si=l9ea08X28Ov2CajR)

## 🎮 Game Highlights 🎮
1. 🐦 **Birds with Unique Abilities**  
   Each bird has a special ability (e.g., acceleration, splitting, explosion, etc.).

2. 🐷 **Animated Birds and Pigs**  
   Smooth animations for flight, impact, and destruction to enhance visual feedback.

3. 🧱 **Manifold Collision Shapes**  
   Structures, pigs, and birds use different shapes for realistic collisions and interactions.

4. 💨 **Smoke & Particle Effects**  
   Visual effects like explosions, debris, and smoke trails for added polish.

5. 🎥 **Dynamic Camera**  
   Follows the bird in flight and zooms in/out to keep the action in view.

6. 🔊 **Sound Integration**  
   Background music and sound effects for launches, collisions, destruction, and bird abilities (powered by irrKlang).


## 🎮 How to Play 🎮
-  **Drag & Release** – Use **left-click** to stretch the slingshot and launch the bird.  
-  **Right-Click** – Activate the bird’s **special ability** mid-air.  
- **Press 1–6** -  Load levels with different birds:  
    - 1\. 🟥 Red Bird  
    - 2\. 💛 Yellow Bird  
    - 3\. 🔵 Blue Bird  
    - 4\. ⚫ Black Bird  
    - 5\. ⚪ White Bird  
    - 6\. 🟢 Green Bird
  
-  **Press the same number again** to **restart** the current level.
-  **Toggle Debug Mode** – Press **P** to enable and **L** to disable debug mode.

---

### The documentation is divided into two main parts:

- The first part covers the physics behind the Angry Birds components, including collisions, bird abilities, particles, and related concepts.

- The second part provides a brief overview of the mainly gameobject code architecture used in creating this demo.


## 📘 Index

1. Physics
   - [Box2D](#a-box2d)
   - [Collisions in Box2D](#b-collisions-in-box2d)
   - [Birds](#c-birds)
   - [Particles](#d-particles)
2. Code Architecture
---

# 🤓 PHYSICS OF ANGRY BIRDS 🤓
This section covers the key physics components used in the game, such as bird abilities, particle effects, and the role of the Box2D library in driving the simulation.

Before exploring the specific physics mechanics of Angry Birds, it’s important to first grasp the fundamentals of the Box2D physics engine.

## (a) Box2D 
This project uses **Box2D 2.32**, a 2D physics engine written in C++ by Erin Catto. Newer versions of Box2D are written in C, which can also be explored if preferred. Box2D provides realistic simulation of rigid body dynamics, gravity, collisions, forces, friction, restitution (bounciness), and joints—key for replicating Angry Birds' gameplay.

### Box2D World

All physics interactions occur within a single Box2D world that handles gravity, collisions, and object behaviors. The physics update runs at a fixed timestep of 16.6 ms (60 FPS) for smooth and consistent simulation.

#### World Settings

- **Gravity:** Set to -10.0f along the Y-axis to simulate real-world gravity.
- **Scale:** 50 pixels = 1 meter (Box2D uses meters). This scaling ensures consistent physics behavior relative to the game's visuals.

Physics calculations drive the position and rotation of game objects, updated every frame in the game loop.

### Box2D Simulation Steps

Each frame, the physics update follows this sequence:

1. **Time Step:** Usually 1/60 seconds for 60 FPS.
2. **World Step:** Advances the physics simulation (`World::getWorld()->Step()`), handling collisions and movement.
3. **Update Objects:** Retrieves each body's position and angle to update corresponding game objects.
4. **Iterate Bodies:** Loops through all physics bodies to ensure accurate state updates.

![image](https://github.com/user-attachments/assets/64644d6f-ffc1-477a-adc6-52bb73efd52a)


### Creating Physics Bodies

A physics body represents an object affected by forces and collisions. To create one, define:

1. **Body Definition (`b2BodyDef`):** Sets initial position, rotation, and type:
   - *Static:* Immovable (e.g., walls).
   - *Dynamic:* Movable, affected by physics (e.g., birds, pigs).
   - *Kinematic:* Moved manually, unaffected by forces (e.g., moving platforms).

2. **Shape Definition:** Defines the collision boundaries with shapes like:
   - `b2PolygonShape`
   - `b2CircleShape`
   - `b2EdgeShape`

3. **Fixture:** Attaches a shape to a body and defines properties such as:
   - Density (mass)
   - Friction (surface sliding)
   - Restitution (bounciness)

4. **Position & Rotation:** Set initially in meters and radians via the body definition.

Here is a basic example of creating a physics body for a rectangular block matching the sprite size:

![image](https://github.com/user-attachments/assets/de55993b-1370-4228-826a-2519e0a28a1a)


For detailed Box2D documentation, see [Box2D official docs](https://box2d.org/doc_version_2_4/).


## (b) Collisions in Box2D

In Box2D, collisions are central to simulating realistic interactions between objects. When two physics bodies (like birds and blocks) come into contact, Box2D calculates forces and responses based on their physical properties.

### 🔑 Key Concepts 

1. **Collision Shapes**  
   Each body is attached to a shape (circle, polygon, edge, etc.) which defines its collision boundary.

2. **Fixtures**  
   Fixtures connect shapes to bodies and define physical properties such as:
   - **Density** – Affects mass
   - **Friction** – Affects sliding resistance
   - **Restitution** – Affects bounciness

3. **Collision Detection**  
- Box2D first filters out objects that are too far apart to collide. Then, for objects that are close enough, it performs precise checks to find exact contact points and calculate collision details needed for realistic interactions.

4. **Collision Callbacks**  
   Custom collision responses are handled through:
   - `BeginContact()`: Called when contact starts
   - `EndContact()`: When contact ends
   - `PreSolve()`: Allows modifying contact before it resolves
   - `PostSolve()`: Used to analyze the results (like impulses)

---

### 🧩 Custom Collision Handling

In this project, I’ve implemented a custom collision system using a class `AngryBirdsContactListener`, derived from `b2ContactListener`. It allows me to define custom behaviors when specific objects interact (e.g., birds hitting blocks or pigs).

To handle specific outcomes depending on object types, I’ve used the **Visitor Pattern**. This enables defining different behavior depending on both objects in a collision — for example, a Blue Bird colliding with a Glass Block vs. a Stone Block.

---

### 🛠️ Collision Processing

To make the physics in this game feel real and responsive, every object that takes part in a collision — whether it's a bird, a block, or a pig — needs a few things:
- A **physics body** (which can be either static or dynamic)
- A **health value**
- A **material type**, like wood, glass, or stone if a block

When two objects collide, Box2D handles the physics and gives us detailed information about what happened during the impact. This info is what I use to figure out how much damage each object should take.

---

### 📊 Using Impulse to Calculate Damage

One of the most important things Box2D gives us during a collision is the **normal impulse** — basically, how hard two objects hit each other.

We also get:
- **Tangential impulse**, which tells us how much sliding happened
- **Contact points**, which show exactly where the objects touched

But it’s the **normal impulse** that really matters when it comes to applying damage. The harder the hit, the bigger the impulse — and the more damage we apply.

---

### 🟦 Example: Blue Bird Hits a Block

Let’s walk through a common scenario: a **Blue Bird** hits a **block**. Different block types take damage differently:
- **Glass** breaks easily, so it has a **high damage multiplier**
- **Wood** is average
- **Stone** is tough, so it needs a strong hit to crack

When the Blue Bird hits the block, we check:
1. What kind of block it is
2. How strong the collision was
3. Whether that hit was hard enough to cause damage

This makes every bird–block interaction feel unique and satisfying.

---

### 🛠️ How It Works Under the Hood

Here’s the basic flow:

1. **Get Collision Info**
   - I grab data from Box2D (using the `b2WorldManifold`) to understand where and how the collision happened.

2. **Check the Block’s Type**
   - Based on whether it's made of **wood, glass, or stone**, I apply a different damage threshold and multiplier.

3. **Apply Damage**
   - If the collision impulse is big enough:
     - I calculate how much damage to apply.
     - If that damage is greater than the block’s remaining health, I reduce its life or destroy it.
     - For fragile materials like glass, even small impacts can be devastating.

---

### 💻 Sample Code: Bird–Block Collision Handling
![image](https://github.com/user-attachments/assets/82e7438e-d0c2-4a12-bc92-313aa2a3f884) ![image](https://github.com/user-attachments/assets/86cddf86-e9f3-4fbf-aa9d-0bff70640f44)




### 🎯 Why This Matters

This system makes the game feel more realistic and fun. Instead of every block behaving the same way, the **material type** and **impact strength** really affect the outcome. A bird might bounce off a stone wall, but completely shatter a glass one.


## (c) Birds

In this project, I’ve included **six different birds**, each with their own unique ability. But before I jump into their individual powers, I want to explain how the **slingshot** works—since every bird’s flight starts from there.

### 🏹 How the Slingshot Works

1. I store the **starting position** of the slingshot.
2. When the player drags the bird, I calculate the vector between the current bird position and the slingshot’s starting point.
3. I **normalize** this vector (to get just the direction).
4. Then, I multiply it by the bird’s **individual speed multiplier** to control how fast that bird launches.
Why use a multiplier? Because birds differ in size and weight. Even though the distance pulled is the same for every bird, the resulting speed needs to vary to reflect their physics correctly. The multiplier makes sure the launch feels balanced for each bird.

Here’s a sample code for the force applied when bird is launched:
![image](https://github.com/user-attachments/assets/8a4c202f-5a7d-479f-a46e-aad9d4e0c599)

### Birds Overview
Every bird in the game needs to be defined with a few key physical properties: its **shape**, **speed**, and **size**. These factors directly influence how much **force** the bird applies when it collides with other objects like blocks or pigs.

I use Box2D to set up the physics body for each bird with the following components:

- A **dynamic body** – so the bird responds to forces like gravity and collisions.
- A **b2CircleShape** – representing the bird’s circular collision boundary.
- **Density** – which affects the bird's mass.
- **Friction** – determining how much it slides along surfaces.
- **Restitution** – controlling how bouncy the bird is on impact.

Together, these elements make sure each bird behaves differently based on its physical profile—giving a sense of weight, momentum, and realism during gameplay.

Here is a sample code for setting up the physics body of any bird:
![image](https://github.com/user-attachments/assets/702206b5-40d5-4459-ab53-4bd1b26db5f0)


#### 🟥 (1) Red Bird

The Red Bird is the simplest of all—it doesn't have any special abilities. You just launch it and let physics do the rest.
Here are the physics properties used:

- **Body Type:** `b2_dynamicBody` (affected by gravity and forces)
- **Shape:** `b2CircleShape` centered on the sprite
- **Radius:** Matches the visual size of the bird, converted from pixels to meters
- **Density:** `10.0f` (affects mass)
- **Friction:** `0.3f` (surface interaction)
- **Restitution:** `0.3f` (controls bounciness)
- **Initial Speed Multiplier:** `100.0f` (used to calculate launch velocity from slingshot)

#### Demo:
![RedBirdGif](https://github.com/user-attachments/assets/8ed4ed8c-aeb8-4092-bad3-6368da22e20e)

#### 💛 (2) Yellow Bird
The Yellow Bird is built for speed. It has a moderate density and friction, but what sets it apart is its dash ability. When activated mid-air, it quickly accelerates forward, making it perfect for smashing through wood. Its default speed is already high, and the dash gives it an extra velocity boost.

Here are the physics properties used (_These values are arbitrary, applied based on what gives a better feel, based on rigorous testing_):

- **Body Type:** `b2_dynamicBody` (affected by gravity and forces)
- **Shape:** `b2CircleShape` centered on the sprite
- **Radius:** Matches the visual size of the bird, converted from pixels to meters
- **Density:** `12.0f` (affects mass)
- **Friction:** `0.3f` (surface interaction)
- **Restitution:** `0.3f` (controls bounciness)
- **Initial Speed Multiplier:** `270.0f` (used to calculate launch velocity from slingshot)

#### Special Ability:
For special ability of this bird, I apply an addtional linear impulse.

Sample Code for Special Ability:
![image](https://github.com/user-attachments/assets/96ecd763-40b0-4454-b0ac-9a4e25efcbc6)

#### Demo:
![YellowBirdGif](https://github.com/user-attachments/assets/fe19cdfc-1ca8-4f2a-a1db-1b4cc0ac539a)

#### 🔵 (3) Blue Bird  
The Blue Bird is small and lightweight, with lower density and a smaller radius. Its unique ability allows it to split into three smaller birds mid-air when the special action is triggered. This makes it ideal for taking down fragile structures like glass by spreading impact across multiple points.
I add a damage multiplier when collided with a glass object, as explained in [Collisions in Box2D](#b-collisions-in-box2d)

Here are the physics properties used (_These values are arbitrary, applied based on what gives a better feel, based on rigorous testing_):

- **Body Type:** `b2_dynamicBody` (affected by gravity and forces)
- **Shape:** `b2CircleShape` centered on the sprite
- **Radius:** Matches the visual size of the bird, converted from pixels to meters
- **Density:** `5.0f` (affects mass)
- **Friction:** `0.3f` (surface interaction)
- **Restitution:** `0.3f` (controls bounciness)
- **Initial Speed Multiplier:** `35.0f` (used to calculate launch velocity from slingshot)

#### Special Ability:
Here's how it works:
- I spawn **two additional Blue Birds**, one slightly above and one slightly below the original.
- All three birds are given the **same linear velocity**, so they continue flying in the same forward direction.
- To achieve the spread effect, I apply a **linear impulse** to the two new birds in **opposite vertical directions**—one upwards and one downwards.
- I also flag each new bird with `isSpecialAbilityDone = true` to ensure the special action doesn’t trigger again.
- Their state is set to `LAUNCHED`, and a debug color is assigned to help distinguish them visually during testing.

Sample code for Special Ability:
![image](https://github.com/user-attachments/assets/56fbef81-d959-438b-b275-ad0aa6a0ee75)


#### Demo:

![BlueBirdGif](https://github.com/user-attachments/assets/87d3c850-0ae1-44d7-b775-0a742761fca8)


#### 💣 (4) Black Bird
The Black Bird is all about power. Slower but heavier, it causes massive damage on impact. Its special ability triggers an explosion, perfect for destroying strong materials like stone.

Here are the physics properties used (_These values are arbitrary, applied based on what gives a better feel, based on rigorous testing_):

- **Body Type:** `b2_dynamicBody` (affected by gravity and forces)
- **Shape:** `b2CircleShape` centered on the sprite
- **Radius:** Matches the visual size of the bird, converted from pixels to meters
- **Density:** `5.0f` (affects mass)
- **Friction:** `0.3f` (surface interaction)
- **Restitution:** `0.3f` (controls bounciness)
- **Initial Speed Multiplier:** `120.0f` (used to calculate launch velocity from slingshot)
  
#### Special Ability:
Here’s how the Black Bird’s ability works:

- When activated, it spawns a **bomb particle emitter** (details covered in the particle section).
- Over **100 small explosive particles** are released with force, damaging everything in their path (visualize in the DEMO).
- Special Ability, can be activated on command or when it collided the bird changes color and in the end explodes.

This makes the Black Bird perfect for clearing out dense, reinforced structures.

Sample Code for Emitting Bomb Particles with Smoke:
![image](https://github.com/user-attachments/assets/5b8a8cd6-05ac-4027-b418-1dffe829478c)

#### DEMO:
![BlackBirdGif](https://github.com/user-attachments/assets/50c7fafd-a276-4bbc-bbfd-4d6e0980e9d6)



#### ⚪ (5) White Bird
The White Bird is designed to attack from above by dropping an explosive egg directly below its flight path. When the special ability is triggered, it releases the egg mid-air and then changes color to indicate the ability has been used. 

Here are the physics properties used (_These values are arbitrary, applied based on what gives a better feel, based on rigorous testing_):

- **Body Type:** `b2_dynamicBody` (affected by gravity and forces)
- **Shape:** `b2CircleShape` centered on the sprite
- **Radius:** Matches the visual size of the bird, converted from pixels to meters
- **Density:** `5.0f` (affects mass)
- **Friction:** `0.3f` (surface interaction)
- **Restitution:** `0.3f` (controls bounciness)
- **Initial Speed Multiplier:** `210.0f` (used to calculate launch velocity from slingshot)
  
#### Special Ability:
Here’s how the White Bird’s ability works:

- On activation, the White Bird spawns an **egg** directly beneath it.
- The bird receives a rapid **upward impulse**, making it fly.
- The egg behaves like a black bird: when it **collides**, it **explodes**.
- Over **100 explosive particles** are generated, scattering with force and dealing area damage to nearby objects (visualized in the DEMO).

Sample Code for White Bird Special Ability:
![image](https://github.com/user-attachments/assets/ae87235c-2464-47fb-a6b3-c7fbacebec36)

#### DEMO:

![WhiteBirdGif2](https://github.com/user-attachments/assets/6623b56c-5a07-4739-9f0c-da9571ae3038)


#### 🟢 (6) Green Bird
The Green Bird is unique for its boomerang-like behavior. Unlike other birds, it doesn't just fly straight—it can reverse direction mid-air. This allows it to curve back toward targets, especially useful for hitting hard-to-reach places from behind.

Here are the physics properties used (_These values are arbitrary, applied based on what gives a better feel, based on rigorous testing_):

- **Body Type:** `b2_dynamicBody` (affected by gravity and forces)
- **Shape:** `b2CircleShape` centered on the sprite
- **Radius:** Matches the visual size of the bird, converted from pixels to meters
- **Density:** `5.0f` (affects mass)
- **Friction:** `0.3f` (surface interaction)
- **Restitution:** `0.3f` (controls bounciness)
- **Initial Speed Multiplier:** `150.0f` (used to calculate launch velocity from slingshot)
  
#### Special Ability:
Here’s how the Green Bird’s ability works:

- On activation, the Green Bird will start moving like a boomerang.
- A **force vector** `(-35.0f, -8.0f)` is applied, scaled by the bird’s mass, creating a sudden shift in direction, making it go in opposite direction of it movement.
- At the same time, the bird’s angle is **manually rotated** backward by `0.15 radians` to enhance the turning effect.

Sample Code for Green Bird Special Ability (This is applied in the update function, so changing it velocity gradually):
![image](https://github.com/user-attachments/assets/abd2e896-04f3-4c57-adf4-d8719f1e22c8)


#### Demo:

![GreenBirdGif](https://github.com/user-attachments/assets/115d027b-e681-4c8a-a34f-8d5a6a052d00)


## (d) Particles

Particles in the game refer to visual effects like smoke trails, block destruction debris, feathers, and bomb explosions. This section explains the basic structure and behavior of different types of particles, focusing on **feathers**, **bomb particles**, and **smoke trails**.

### 🔧 Particle System Structure

The particle system is built around two main components:

1. **Particle**  
   Each particle has:
   - A sprite for visual appearance
   - A `maxLifetime` which defines how long it stays active
   - A physics body with properties like size, velocity, and sometimes mass or density
   - Behavior that depends on its type (e.g., feathers float, bomb particles exert force)

2. **Particle Emitter**  
   Acts as a factory that spawns particles at a given location (`Transform`). It can generate one or many particles and can vary the sprite or behavior based on the type of effect being created.

### 🪶 Feather Particles

Feather particles are spawned when birds collide or get destroyed. They're designed for lightweight visual effects and have the following properties:

- **Zero Density**: No need for mass, since they dont need to effect other objects.
- **No Gravity**: They float without falling.
- **No Rotation**: Fixed rotation gives a stable visual.
- **Low Drag**: Simulates smooth, air-like movement.
- **No Collisions**: Marked as sensors so they don’t affect physics.
- **Small Size**: Radius is minimal for subtlety.
- **Short Lifespan**: They disappear quickly to keep things performant.

Feather particles are purely aesthetic and don’t interact with the environment physically—they just spread in a small radius and fade out over time.

#### 🌀 Feather Particle Emitter (Randomization)

To make feather particles look natural, I randomize key properties when spawning them:

- **Position:** Slight offset in X and Y to spread particles outward.
- **Angle:** Each particle gets a random rotation.
- **Lifetime:** Particles live for a short, varied time.
- **Angular Velocity:** Adds random spin to each feather.

This makes the feathers feel scattered and dynamic.

Here is the sample Code for Feather Emitter randomly spawning feathers:

![image](https://github.com/user-attachments/assets/737a55df-fbee-471e-aac3-4510c804ef35)



### 💣 Bomb Particles

Bomb particles are similar to feathers but behave very differently due to their **physical properties**:

- **Higher Density:** Gives them mass, allowing them to impact and push objects on contact.
- **Increased Restitution:** Makes them bounce on impact, mimicking explosive scatter.
- **Physics-Driven:** Unlike feathers, bomb particles affect the environment—knocking over blocks or damaging structures.

These are used in special abilities like the **Black** or **White Bird** explosions to simulate destructive force.

#### 💥 Radial Bomb Particle Emission

When bomb particles are spawned (e.g., during the **Black Bird** explosion), I emit them in **all 360° direction** using a radial force system:

- Each particle is given a **direction vector** based on its angle in a full circle (`0°–360°`).
- A constant **strength** value (e.g., `30.0f`) controls the magnitude of the force.
- All particles are evenly spaced in direction but move outward with the same speed, creating a uniform explosive effect.

Here is the sample Code for Bomb Emitter spawning bomb particles:

![image](https://github.com/user-attachments/assets/7f706126-175f-4de2-8f72-55e1369d466c)

---
### 💨 Bird Smoke Trails

Bird Smoke trails are visual markers that follow birds during their flight, used to show the path taken through the air. Unlike other particles, they don’t interact with physics and are purely for visual feedback.

- **No Physics Body**: Trails are non-collidable and don't affect gameplay.
- **Long Lifespan**: Each trail has a large lifetime, so it stays visible in the scene until explicitly removed.
- **Bird-Exclusive**: Each trail is tied to a single bird.
- **Managed with a Trail Manager**: I maintain exactly **two trails** at any time using a simple queue logic.

#### Behavior:
- A new smoke trail is spawned **every time the bird travels a certain distance** after launch.
- When a new bird is launched, the previous trail remains until the new bird collides.
- Each **trail instance smoke** is recorded in a **trail linked list**, so I can delete the whole trail when the next bird collides.
- The system ensures a clean and minimal visual trail history, always limited to two.


#### Simple smoke particles work much like feather particles but use different sprites. They appear for visual effects such as when a pig is killed or to signal explosions, like those from the black bird or the egg blast.

# 👨‍💻 CODE ARCHITECTURE OF ANGRY BIRDS 👨‍💻

The code architecture of this Angry Birds project is based on object-oriented principles. I won’t be able to dive into every single detail, but I’ll do my best to explain it as clearly as possible.

I’ll start by giving an overview of my approach, breaking down the architecture into sections and focusing on key objects like birds and blocks. I’ll also cover mainly what design patterns that I have used and what purpose, explain the GameObject structure, and how I differentiate between physics bodies and simple props like backgrounds.

To help make things clearer, I’ll include UML diagrams and some sample code snippets to illustrate the logic.

## 🎨 Sprite Rendering Architecture

Our engine draws 2D sprites using three main components: **textures**, **images**, and **sprites**, all managed by dedicated systems.

### Key Components:

1.  **Textures:** Full sprite sheets loaded into memory. Managed by a **Texture Manager**.
2.  **Images:** Defined regions (cut-offs) from textures (e.g., `greenbird_idle`). Managed by an **Image Manager**.
3.  **Sprites:** Visual objects combining an **Image** reference with rendering properties (position, angle, scale). Managed by a **Sprite Node Manager**.

### Sprite Class Hierarchy:

* **`SpriteBase`:** The foundational class for all sprites.
* **`SpriteGame`:** For in-game visuals (e.g., backgrounds). References an **Image**.
* **`SpriteDebugBase`:** Base for debug visualization sprites (e.g., wireframes).
    * **`SpriteDebugClass`es:** Specific shapes like `DebugCircle`, `DebugBox` for physics body visualization.

All sprite types are managed by the **Sprite Node Manager**.

![image](https://github.com/user-attachments/assets/ef8c324a-95df-40c8-a319-025e68e21fe8)


## 🎮 Game Object Architecture

Game objects populate the world, differentiated by whether they interact with physics or are static props.

### Game Object Hierarchy:

* **`GameObjectBase`:** Base class for all objects. Points to a **`SpriteGame`** for visuals. Used for static props (e.g., backgrounds).
* **`GameObjectPhysics`:** Derived from `GameObjectBase` for physics-enabled objects. Includes a **`SpriteDebug`** reference to visualize its physics body (e.g., Box2D). Specific physics behaviors are handled in derived classes like `Bird_Red`.

### Game Object Management:

* **`GameObjectNode`:** Wraps heterogeneous game objects for consistent management.
* **`GameObjectManager`:** Central manager for creating, storing, and updating all game objects within the engine's main loop.

### UML Diagram: Game Object Class Hierarchy
![image](https://github.com/user-attachments/assets/f0143575-31e3-47ec-8ce0-a4fe1e9a00f2)

## 🔒 Memory Management & Principles

**Memory leakage prevention** is crucial. All components (Textures, Images, Sprites, Game Objects) are handled by dedicated **Managers**. These managers centralize loading, provide efficient access, and are responsible for proper memory cleanup when objects are removed, ensuring a stable and performant game.



## 📐 Design Patterns Implemented

To keep my game modular, maintainable, and efficient, I’ve implemented several classic design patterns. These help me manage complexity, improve performance, and organize responsibilities clearly across systems.


### 1. 🧭 Singleton Pattern

**Used in:**  
- `TextureManager`, `ImageManager`, `SpriteNodeManager`, `GameObjectManager`, `TimerEventManager`, `SoundManager`, etc.

**Why I use it:**  
Each manager needs to exist only once during the game’s lifetime. The Singleton pattern gives me a global and consistent way to access systems like texture loading, sprite updates, and event handling — without worrying about duplicate instances.

---

### 2. 🔁 Object Pooling

**Why I use it:**  
Creating and destroying objects repeatedly is expensive. Instead, I reuse them from a pool. When an object (like a bird or a block piece) is no longer needed, I return it to the pool instead of deleting it. This improves performance, reduces memory overhead, and keeps the engine fast — especially during heavy scenes.

---
### 3. 🏭 Factory Pattern

**Used in:**  
- Spawning game objects like Birds, Blocks, and Pigs.

**Why I use it:**  
The Factory pattern lets me centralize and simplify the creation of complex GameObjects. I pass a type (like `"RedBird"` or `"Block_Wood"`), and the factory gives me the object with the correct physics, sprites, and behaviors. This keeps creation clean, consistent, and flexible for future additions.

---

### 4. 🎮 Command Pattern

**Used in:**  
- Timer Events  
- Animation triggers  
- Encapsulated game logic

**Why I use it:**  
Commands allow me to turn a specific action into an object — like “start animation” or “remove particle.” I can queue, delay, or cancel these actions easily without tightly coupling the logic. It also helps cleanly separate behavior across systems, like input, physics, and rendering.

---

### 5. ⏱️ Timer Event System

**Used for:**  
- Particle lifetime cleanup  
- Delayed bird actions  
- Time-based logic (e.g., exploding after a delay)

**Why I use it:**  
I use a timer event system to handle delayed logic. It schedules a command with a countdown. Once time runs out, the command executes.  
For example:
- A **trail** fades after flying a distance.
- A **bomb bird** explodes after a delay.
- A **feather particle** despawns after some time.

This system helps me build dynamic, time-based behaviors without hardcoding delays or cluttering logic with timers everywhere.

---
### 6. 🧾 Visitor Pattern

**Used in:**  
- Collision handling between all GameObjects (Birds, Pigs, Blocks, etc.)

**Why I use it:**  
In a game like Angry Birds, many objects interact differently when they collide — a bird hitting a pig behaves differently than a pig hitting a block.  
The Visitor Pattern helps me handle these collisions without messy `if`-else chains or type-checking everywhere.
Each GameObject implements an Accept method and passes in the colliding object as a visitor. That visitor then calls the correct Visit method based on type.

These patterns work together to give me a clean, reliable architecture. The goal is always to keep systems decoupled, code reusable, and performance predictable — especially for a physics-heavy game like Angry Birds.


#### This is just me giving you the gist of all the patterns that I have used for various elements of the game. I won't be able to explain everything in this document, but hope this will make you understand the basics of my code architecture.

