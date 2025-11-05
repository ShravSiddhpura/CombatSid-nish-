# CombatSid-nish-

Here is the project document for your "3D Combat Arena" game, tailored to your AI syllabus.

-----

# AI Project Document: 3D Combat Arena

**Subject:** Artificial Intelligence
**Project:** 3D Combat Arena (A 3D Shooter with Goal-Based Agents)

-----

## 1\. The Game: Introduction

**3D Combat Arena** is a real-time, 3D arena shooter built using **Three.js** and JavaScript (ES Modules). The player controls a cyan-colored humanoid character from a 3/4 top-down (isometric-style) perspective.

The core of the game is a "survival" scenario where the player is placed in a static, walled arena with four hostile AI-controlled enemies (red). The AI agents are designed to actively hunt and shoot at the player, while the player must use movement and their own weapon to eliminate all enemies and win.

-----

## 2\. Game Rules & Objectives

  * **Controls:**
      * **W / A / S / D:** Move the player character.
      * **SPACE:** Shoot a projectile in the direction the player is facing.
      * **P:** Pause or resume the game.
  * **Winning:** The player wins by successfully shooting and eliminating all four red enemies.
  * **Losing:** The player loses if they are hit by an enemy projectile.
  * **Environment:**
      * The arena is enclosed by outer walls that block both movement and projectiles.
      * Internal walls act as obstacles that the player and AI must navigate.
      * Projectiles (from player or AI) are destroyed upon hitting a wall.

-----

## 3\. Technology Used

  * **Core Language:** JavaScript (ES6 Modules)
  * **Rendering Engine:** **Three.js** (v0.155). This library manages:
      * The scene, lighting (`AmbientLight`, `DirectionalLight`), and `PerspectiveCamera`.
      * All 3D models (characters, walls, bullets) which are created from simple geometries (`BoxGeometry`, `SphereGeometry`, `PlaneGeometry`).
      * Materials (`MeshPhongMaterial`, `MeshBasicMaterial`) for color and lighting.
  * **Game Logic:**
      * **State Management:** Simple boolean flags (`gamePaused`, `gameStarted`, `playerAlive`) control the game flow.
      * **UI:** The Start, Pause, and Game Over menus are standard HTML/CSS `div` elements overlaid on the 3D canvas.
      * **Physics:** Collision detection is handled manually by checking an object's position against the `walls` array (`collides()` and `bulletHitsWall()` functions).

-----

## 4\. AI Algorithm Analysis

The AI logic governs the behavior of the four red "enemy" agents. Each agent runs its logic independently on every frame of the game.

### 4.1. PEAS Representation

The AI in this project can be described using the **PEAS (Performance, Environment, Actuators, Sensors)** framework:

  * **Performance Measure:**
    1.  **Hit Player:** Successfully land a projectile on the player.
    2.  **Pursue Player:** Minimize the distance to the player to maintain engagement.
    3.  **Survive:** (Implicitly) The AI does not have survival logic (e.g., dodging). Its main performance is offensive.
  * **Environment:**
      * **Fully Observable:** The AI has direct access to the exact coordinates of the player (`player.position`) and all walls (`walls` array) at all times.
      * **Dynamic:** The environment changes as the player and other agents move.
      * **Continuous:** All agents move in continuous 3D space (not a grid).
      * **Multi-agent:** The environment contains the player and other AI agents (though AI agents do not interact with each other).
  * **Actuators:**
      * `e.position`: Modifying its 3D coordinates (movement).
      * `e.lookAt()`: Changing its rotation to face the player.
      * `enemyShoot()`: Spawning a bullet projectile.
  * **Sensors:**
      * `player.position`: (Directly read from the game state).
      * `walls` array: (Used to check for collisions via the `collides()` function).

### 4.2. Agent Type: Goal-Based Agent

This AI is a **Goal-Based Agent**. It is more advanced than a Simple Reflex Agent because its actions are based on a "goal."

  * **Goal:** The agent's goal is static and explicit: **"Be at the player's current location."**
  * **Action:** All its actions are taken to reduce the difference between its current state (`e.position`) and its goal state (`player.position`).

It is *not* a Model-Based Agent, as it doesn't predict the player's *future* position. It only reacts to their *current* position.

### 4.3. Core Algorithm: Greedy Search & Hill Climbing

The AI's logic is split into **Targeting** and **Pathfinding**.

**1. Targeting Logic:**
The targeting is a simple reflex.

```javascript
// AI looks directly at the player
e.lookAt(player.position); 

// AI calculates a firing vector to the player's *current* position
const dir = player.position.clone().sub(enemy.position).normalize();
```

This is **linear trajectory** targeting. It shoots where the player *is*, not where they *will be*.

**2. Pathfinding (Movement) Logic:**
This is the most relevant part of the AI. The agent uses a **Greedy Search** algorithm to move.

```javascript
// 1. Get direction to the goal (player)
const dir = player.position.clone().sub(e.position).normalize();
// 2. Calculate the next position based on this "greedy" move
const next = e.position.clone().add(dir.multiplyScalar(enemySpeed * delta));
// 3. Only execute the move if it's "safe" (not in a wall)
if (!collides(next)) e.position.copy(next);
```

This algorithm is a perfect example of **Hill Climbing**:

  * **"Landscape":** The 3D arena.
  * **"Height":** The "goodness" of a position, defined as its *negative distance* to the player. The "hilltop" (goal) is at the player's position, where the distance is 0.
  * **Algorithm:** The AI always attempts to move in the direction of "steepest ascent" (the most direct, straight-line path "uphill" toward the player).
  * **The Flaw (Local Optimum):** This AI will get stuck on **local optimums**. If a wall is between the AI and the player, the "uphill" direction is *through* the wall. The `collides()` check stops the AI from moving. Because the AI cannot "see" any other path (like moving "downhill" temporarily to go *around* the wall), it gets stuck, perpetually trying to move through the obstacle.

-----

## 5\. Connection to AI Syllabus

  * **Agents and Environments:** This project is a clear implementation of a **Goal-Based Agent** in a **Fully Observable, Dynamic, Multi-agent** environment.
  * **Problem Solving Agent:** The AI is a problem-solver where:
      * **State:** The AI's (x, z) position.
      * **Goal:** The player's (x, z) position.
      * **Action:** Move one step in the straight-line direction of the goal.
  * **Search Methods (Informed):** The pathfinding is a **Greedy Best-First Search**.
      * **Heuristic (h(n)):** The heuristic is the **Euclidean distance** to the player. The AI *greedily* chooses the action (moving in a straight line) that it believes will minimize this heuristic to zero.
  * **Local Search (Hill Climbing):** This is the *best* description of the AI's movement. It's a Hill Climbing algorithm that seeks to maximize "closeness" (minimize distance). Its primary, observable flaw is its inability to escape **local optimums** (getting stuck on walls).
  * **Adversarial Search:** This project **does not** use Adversarial Search. The AI does not use Minimax or Alpha-Beta Pruning. It does not model the player as an "adversary" or predict the player's future moves; it only reacts to the player's current, known position.

-----

## 6\. Algorithm Complexity

The computational load of the AI is part of the main `animate` loop.

Let:

  * **E** = Number of enemies
  * **W** = Number of walls
  * **Pb** = Number of player bullets
  * **Eb** = Number of enemy bullets

The complexity of the **AI's decision-making (thinking)** per frame is analyzed as follows:

The main "Enemy logic" block iterates `E` times (once per enemy).

```javascript
enemies.forEach((e) => {
  // ...
  const dir = player.position.clone().sub(e.position).normalize(); // O(1)
  const next = e.position.clone().add(dir.multiplyScalar(enemySpeed * delta)); // O(1)
  
  if (!collides(next)) ... // This is the dominant cost
  // The collides() function loops through all 'W' walls
  // So, collides() has a complexity of O(W)
  
  e.lookAt(player.position); // O(1)
});
```

  * The complexity for **one** enemy to make a move decision is **$O(W)$**, as it must check its next position against all walls.
  * Therefore, the total complexity for *all* AI agent decisions per frame is **$O(E \times W)$**.

This is extremely efficient. Since `E` (4) and `W` (7) are small, fixed numbers, the AI's "thinking" time is negligible and scales linearly, making it perfect for real-time execution.

-----

## 7\. Sample Game Output

**Screenshot 1: The Arena**

*Description: The game starts. The player (cyan) is positioned at the bottom, and the four AI agents (red) are spread across the arena.*

**Screenshot 2: AI Pathfinding (and Flaw)**

*Description: The Hill Climbing algorithm's flaw is visible. The AI (right) is trying to reach the player (left), but the wall is a "local optimum." The AI will not move around it and is "stuck."*

**Screenshot 3: Game Over**

*Description: The player was hit by a projectile from an AI agent, resulting in a "Game Over" state.*

**Screenshot 4: Victory**

*Description: The player has successfully eliminated all AI agents, achieving the win condition.*
