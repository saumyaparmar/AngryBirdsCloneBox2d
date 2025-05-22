# 🐦 Angry Birds Clone – Built with Box2D 🐦


This is a clone of the Angry Birds game, built using the Box2D 2.3.2 physics engine and developed in C++. The project replicates the core gameplay mechanics and physics behavior of the original game as closely as possible. It was created as part of an independent study project for my graduate program.

The game runs on a custom 2D engine developed by my professor, which provides support for sprite rendering, debug wireframe drawing, and camera functionality. Sound integration is handled using irrKlang. 
The game is designed for a resolution of 1280x720; fullscreen mode is not supported due to input positioning issues.

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

  &nbsp;&nbsp;&nbsp;&nbsp;1. 🟥 Red  
  &nbsp;&nbsp;&nbsp;&nbsp;2. 💛 Yellow  
  &nbsp;&nbsp;&nbsp;&nbsp;3. 🔵 Blue  
  &nbsp;&nbsp;&nbsp;&nbsp;4. ⚫ Black  
  &nbsp;&nbsp;&nbsp;&nbsp;5. ⚪ White  
  &nbsp;&nbsp;&nbsp;&nbsp;6. 🟢 Green
  
-  **Press the same number again** to **restart** the current level.
-  **Toggle Debug Mode** – Press **P** to enable and **L** to disable debug mode.

---

### The documentation is divided into two main parts:

- The first part covers the physics behind the Angry Birds components, including collisions, bird abilities, particles, and related concepts.

- The second part provides a brief overview of the code architecture used in creating this demo.


## 📘 Index

1. Physics
   - [Box2D](#a-box2d)
   - [Collisions in Box2d](#b-collisions-in-box2d)
   - [Birds](#birds)
   - [Particles](#particles)

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
- A **material type**, like wood, glass, or stone

When two objects collide, Box2D handles the physics and gives us detailed information about what happened during the impact. This info is what we use to figure out how much damage each object should take.

---

#### 📊 Using Impulse to Calculate Damage

One of the most important things Box2D gives us during a collision is the **normal impulse** — basically, how hard two objects hit each other.

We also get:
- **Tangential impulse**, which tells us how much sliding happened
- **Contact points**, which show exactly where the objects touched

But it’s the **normal impulse** that really matters when it comes to applying damage. The harder the hit, the bigger the impulse — and the more damage we apply.

---

#### 🟦 Example: Blue Bird Hits a Block

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

#### How It Works Under the Hood

Here’s the basic flow:

1. **Get Collision Info**
   - We grab data from Box2D (using the `b2WorldManifold`) to understand where and how the collision happened.

2. **Check the Block’s Type**
   - Based on whether it's made of **wood, glass, or stone**, we apply a different damage threshold and multiplier.

3. **Apply Damage**
   - If the collision impulse is big enough:
     - We calculate how much damage to apply.
     - If that damage is greater than the block’s remaining health, we reduce its life or destroy it.
     - For fragile materials like glass, even small impacts can be devastating.

---

### 💻 Sample Code: Bird–Block Collision Handling
![image](https://github.com/user-attachments/assets/82e7438e-d0c2-4a12-bc92-313aa2a3f884) ![image](https://github.com/user-attachments/assets/86cddf86-e9f3-4fbf-aa9d-0bff70640f44)




### 🎯 Why This Matters

This system makes the game feel more realistic and fun. Instead of every block behaving the same way, the **material type** and **impact strength** really affect the outcome. A bird might bounce off a stone wall, but completely shatter a glass one.

