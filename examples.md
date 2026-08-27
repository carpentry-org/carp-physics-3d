# Examples for carp-physics-3d

This file contains usage examples for all sub-modules in the library.

## Geometry

## Raycasting against AABBs

Useful for mouse picking or simple projectile physics.

```carp
(use Geometry)

(let [box (AABB.init (Vector3.init -1.0 -1.0 -1.0) (Vector3.init 1.0 1.0 1.0))
      ray (Ray.init (Vector3.init 0.0 0.0 -10.0) (Vector3.init 0.0 0.0 1.0))]
  (if (Ray.intersect-aabb? &ray &box)
    (IO.println "Ray hit the box")
    (IO.println "Ray missed")))
```

## Sphere-AABB Collision

Standard for character controllers or item pickups.

```carp
(let [player (Sphere.init (Vector3.init 0.5 0.5 0.0) 0.5)
      wall (AABB.init (Vector3.init 1.0 0.0 0.0) (Vector3.init 2.0 1.0 1.0))]
  (if (Sphere.intersect-aabb? &player &wall)
    (IO.println "Player is touching the wall")
    (IO.println "Clear path")))
```

## Plane Math

Projecting a point onto a plane (useful for shadows or sliding physics).

```carp
(let [ground (Plane.from-point-normal (Vector3.init 0.0 0.0 0.0) (Vector3.init 0.0 1.0 0.0))
      p (Vector3.init 10.0 5.0 10.0)
      proj (Plane.project-point &ground &p)]
  (IO.println &(format "Projected point: %s" &(Vector3.str &proj))))
```


---

## Collision

## 1. Simple Sphere-Sphere Check

```carp
(load "carp-collision/collision.carp")
(use Collision)
(use CollisionChecker)

(defn main []
  (let [h1 (Handle.init (Uint64.from-long 0l) (Uint32.from-long 1l))
        v1 (Volume.Ball (Sphere.init (Vector3.init 0.0 0.0 0.0) 1.0))
        c1 (Collidable.init h1 v1 (Uint32.from-long 1l) (Uint32.from-long 1l) false)
        
        h2 (Handle.init (Uint64.from-long 1l) (Uint32.from-long 1l))
        v2 (Volume.Ball (Sphere.init (Vector3.init 1.5 0.0 0.0) 1.0))
        c2 (Collidable.init h2 v2 (Uint32.from-long 1l) (Uint32.from-long 1l) false)]
    
    (match (check-pair &c1 &c2)
      (Maybe.Just (CollisionResult.Physical col)) 
          (println* "Collision! Depth: " @(Contact.depth (Collision.contact &col)))
      _ (println* "No physical collision."))))
```

## 2. Using Layers and Masks (Symmetric)

```carp
(load "carp-collision/collision.carp")
(use Collision)
(use CollisionChecker)

(defn main []
  (let [player-layer (Uint32.from-long 1l)
        enemy-layer (Uint32.from-long 2l)
        projectile-layer (Uint32.from-long 4l)
        
        ;; Projectiles hit enemies
        proj (Collidable.init (Handle.init (Uint64.from-long 0l) (Uint32.from-long 1l)) 
                             (Volume.Ball (Sphere.init (Vector3.init 0.0 0.0 0.0) 0.1)) 
                             projectile-layer enemy-layer false)
        ;; Enemies hit players
        enemy (Collidable.init (Handle.init (Uint64.from-long 1l) (Uint32.from-long 1l)) 
                              (Volume.Box (AABB.init (Vector3.init -1.0 -1.0 -1.0) (Vector3.init 1.0 1.0 1.0))) 
                              enemy-layer player-layer false)]
    
    ;; This will result in Nothing because filtering is symmetric:
    ;; proj.mask (enemy) matches enemy.layer (enemy) -> YES
    ;; BUT enemy.mask (player) DOES NOT match proj.layer (projectile) -> NO
    (match (check-pair &proj &enemy)
      (Maybe.Just _) (println* "Hit!")
      (Maybe.Nothing) (println* "Symmetric filter blocked collision."))))
```

## 3. Integrating with SpatialGrid and Avoiding Duplicates

```carp
(load "carp-collision/collision.carp")
(use Collision)
(use CollisionChecker)
(use SpatialGrid)

(defn main []
  (let [grid (SpatialGrid.new 10.0 10 10 10)
        entities [(Collidable.init (Handle.init (Uint64.from-long 0l) (Uint32.from-long 1l)) 
                                  (Volume.Ball (Sphere.init (Vector3.init 5.0 5.0 5.0) 1.0)) 
                                  (Uint32.from-long 1l) (Uint32.from-long 1l) false)
                  (Collidable.init (Handle.init (Uint64.from-long 1l) (Uint32.from-long 1l)) 
                                  (Volume.Ball (Sphere.init (Vector3.init 6.0 5.0 5.0) 1.0)) 
                                  (Uint32.from-long 1l) (Uint32.from-long 1l) false)]]
    (do
      ;; Sync grid with entities
      (for [i 0 (Array.length &entities)]
        (let [e (Array.unsafe-nth &entities i)]
          (SpatialGrid.insert! &grid &(Collision.get-aabb (Collidable.volume e)) (Uint64.from-long (Int.to-long i)))))
      
      ;; Query collisions for the first entity, using symmetric flag to avoid duplicate pairs in broad loops
      (let [self (Array.unsafe-nth &entities 0)
            hits (query-and-check &grid self &entities true)]
        (println* "Entity 0 hit " (Array.length &hits) " others.")))))
```


---

## Spatial

## Efficient Ray-Grid Traversal

The `query-ray` function uses a 3D DDA algorithm to visit only the cells the ray actually passes through.

```carp
(use SpatialGrid)

(let [grid (new 10.0 10 10 10)
      ray (create-ray &(Vector3.init -5.0 5.0 5.0) &(Vector3.init 1.0 0.0 0.0))]
  (let [hits (query-ray-unique &grid &ray 100.0)]
    ;; 'hits' contains IDs of all objects in the ray's path
    ...))
```

## Neighborhood Queries

Useful for AI, physics, or particle effects where you need to find all objects near a specific point.

```carp
(let [grid (new 5.0 50 50 50)
      explosion-pos (Vector3.init 25.0 25.0 25.0)
      explosion-radius 10.0
      sphere (Sphere.init explosion-pos explosion-radius)]
  (let [victims (query-sphere-unique &grid &sphere)]
    (foreach [id victims]
      (apply-damage @id 100))))
```

## Management in a Game Loop

Standard pattern for a game engine: clear the grid every frame and re-insert active objects.

```carp
(defn update [grid entities]
  (do
    (clear! grid)
    (foreach [e entities]
      (insert! grid (get-aabb e) (get-id e)))))
```


---

## Rigidbody

## 1. Physical Entity Setup
This example shows how a `RigidBody` acts as the single component needed for a physical entity in your world.

```clojure
(load "rigidbody.carp")
(use RigidBody)
(use Vector3)
(use Quaternion)

(defn create-player []
  (let [pos (Vector3.init 0.0 2.0 0.0)
        rot (Quaternion.identity)
        ;; Player: 1.0kg, low bounciness, high friction
        player (RigidBody.new pos rot 1.0 0.2 0.8 0.98)]
    player))
```

## 2. Proxy Accessors
You can interact with the spatial state of the object without reaching into the underlying `transform`.

```clojure
(defn debug-rb [rb]
  (do
    (println* "Current Position: " (Vector3.str (RigidBody.position rb)))
    (println* "Current Velocity: " (Vector3.str @(Body.velocity (RigidBody.body rb))))))
```

## 3. Interaction with Solvers
Because `RigidBody` unifies all data, solvers become much cleaner.

```clojure
(use Solver)

;; Imagine a hypothetical solver call:
(Solver.solve! &rbA &rbB contact)
;; Internally, it can access (RigidBody.transform rbA) and (RigidBody.body rbA) seamlessly.
```


---

## Physics

## 1. Basic Rigid Body Bounce
A simple example of resolving a collision between a moving dynamic body and a static wall.

```clojure
(load "physics.carp")
(use Solver)
(use Transform)
(use Body)
(use Collision)

(defn resolve-impact [trans body static-trans static-body contact]
  (let [h1 (Handle.init 0u64 1u32)
        h2 (Handle.init 1u64 1u32)
        col (Collision.init h1 h2 contact)]
    (Solver.solve! trans body static-trans static-body &col)))
```

## 2. Friction Simulation
Setting up a collision where friction will slow down an object sliding along a surface.

```clojure
(load "physics.carp")
(use Solver)
(use Body)

(defn setup-friction []
  (let [;; Body with 0.5 friction
        b (Body.new 1.0 0.5 0.5 0.9)
        ;; Static floor with 0.8 friction
        floor (Body.static 0.5 0.8)]
    (do
      (println* "Friction pair established."))))
```

## 3. The Iterative Solver Loop
For stable stacks and multi-point contact, it is recommended to run the solver in multiple iterations per frame.

```clojure
(load "physics.carp")
(use Solver)

(defn solve-world [transforms bodies collisions]
  ;; Standard practice: 8 velocity iterations for stability
  (for [iter 0 8]
    (for [i 0 (Array.length collisions)]
      (let [col (Array.unsafe-nth collisions i)
            ;; Retrieve handles and bodies...
            ]
        (Solver.solve! t-a b-a t-b b-b col)))))
```


---

## Dynamics

## 1. Simple Particle with Gravity
This example shows how to set up a basic particle simulation where an object falls under gravity and experiences air resistance (damping).

```clojure
(load "dynamics.carp")
(use Body)
(use Integrator)
(use Transform)
(use Vector3)

(defn main []
  (let [trans (Transform.identity)
        ;; Mass=1.0, Bounciness=0.5, Friction=0.5, Damping=0.9
        body (Body.new 1.0 0.5 0.5 0.9)]
    (while true
      (let [dt 0.016]
        (do
          ;; Apply Gravity
          (Body.apply-force! &body &(Vector3.init 0.0 -9.81 0.0))
          
          ;; Integrate
          (Integrator.step! &trans &body dt)
          
          (println* "Current Position: " (Vector3.str (Transform.position &trans))))))))
```

## 2. Setting up a Static Floor
Static bodies are immune to forces and integration. They are used for the "immovable" parts of your world.

```clojure
(load "dynamics.carp")
(use Body)
(use Transform)
(use Vector3)

(defn setup-world []
  (let [floor-trans (Transform.init (Vector3.init 0.0 -1.0 0.0) 
                                   (Quaternion.identity) 
                                   (Vector3.init 100.0 1.0 100.0))
        floor-body (Body.static)]
    (do
      (println* "Static floor initialized at Y=-1.0"))))
```

## 3. Immediate Impulses
Unlike continuous forces, impulses change velocity instantly. This is perfect for jumps, explosions, or recoil.

```clojure
(load "dynamics.carp")
(use Body)
(use Vector3)

(defn jump [body]
  ;; Add 5.0 units of upward velocity instantly
  (Body.apply-impulse! body &(Vector3.init 0.0 5.0 0.0)))
```

