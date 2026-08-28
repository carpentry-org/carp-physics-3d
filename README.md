# carp-physics-3d

A consolidated Physics Library suite for the Carp programming language.

## Modules Included
- **[Collision](#collision)**: See module documentation below.
- **[Dynamics](#dynamics)**: See module documentation below.
- **[Geometry](#geometry)**: See module documentation below.
- **[Physics](#physics)**: See module documentation below.
- **[Rigidbody](#rigidbody)**: See module documentation below.
- **[Spatial](#spatial)**: See module documentation below.

## Installation

```
(load "git@github.com:carpentry-org/carp-physics-3d@master")
```

That pulls in every module. A single one can be loaded on its own:

```
(load "git@github.com:carpentry-org/carp-physics-3d@master" "collision.carp")
```

## Examples

See [examples.md](examples.md) for module usage examples, and the
[API documentation](https://carpentry.dev/carp-physics-3d) for the full
reference.

---

## Geometry

A lightweight, deterministic collision and intersection library for 3D geometry.  
Supports both **query-based intersection tests** and **contact-based collision resolution**.

---

## Overview

This module provides:

### Ray Queries (non-penetrating)
Used for:
- visibility tests
- picking / selection
- raycasting
- bullet tracing

Returns: `RayHit`

---

### Contact Collisions (penetration-based)
Used for:
- physics resolution
- overlap correction
- broad-phase interaction responses

Returns: `Contact`

---

## Core Types

### Primitives
- `Sphere`
- `AABB`
- `Plane`
- `Segment`
- `Ray`

### Outputs
- `RayHit`
- `Contact`

---

## Coordinate Conventions

### RayHit
- `t` = distance along ray
- `point` = world-space hit position
- `normal` = outward-facing surface normal

### Contact
- `depth` = penetration depth
- `point` = approximate contact point
- `normal` = separation direction (toward second object in most operations)

---

## Key Rules

- All `Ray.direction` and `Plane.normal` vectors are always normalized.
- Segment operations internally use ray casting with clamped distance.
- Degenerate cases (zero-length vectors, overlapping centers) use stable fallback axes.
- EPSILON is used for numerical stability.

---

## Modules

### Geometry

Entry utilities:
- `create-ray`
- `create-plane`

---

### Geometry.Ray

- `at(ray, t)`
- `intersect-sphere(ray, sphere) -> Maybe RayHit`
- `intersect-aabb(ray, aabb) -> Maybe RayHit`

---

### Geometry.Sphere

- `collide-sphere(s1, s2) -> Maybe Contact`
- `collide-aabb(sphere, aabb) -> Maybe Contact`
- `contains?(sphere, point)`

---

### Geometry.AABB

- `collide-aabb(a, b) -> Maybe Contact`
- `contains?(aabb, point)`

---

### Geometry.Plane

- `distance-to-point(plane, p)`
- `project-point(plane, p)`

---

### Geometry.Segment

- `length(segment)`
- `direction(segment)`
- `intersect-sphere(segment, sphere)`
- `intersect-aabb(segment, aabb)`

---

## Design Philosophy

This library prioritizes:

- Determinism (no randomness, no frame dependence)
- Numerical stability over geometric perfection
- Simple primitives over complex manifolds
- Fast rejection before expensive computation
- Clear separation between:
  - intersection queries (`RayHit`)
  - collision resolution (`Contact`)

---

## Performance Notes

- AABB ray tests use slab method (O(1))
- Sphere tests use quadratic solve (O(1))
- Segment tests are ray-reduced (no extra math cost)
- No allocations beyond `Maybe` result construction

---

## Limitations

- No continuous collision manifold generation
- No mesh or triangle support
- No rotational physics (AABB only)
- Contact points are approximations for simplicity

---

## Typical Use Case

- Game physics core
- Simple collision systems
- Ray-based weapon systems
- Spatial queries and triggers

## Examples

See [examples.md](examples.md) for usage examples.


---

## Collision

A modular 3D collision detection and dispatch library for the [Carp](https://github.com/carp-lang/Carp) programming language.

This library bridges the gap between spatial partitioning (`carp-spatial`) and geometric intersection primitives (`carp-geometry`). It provides a high-level API for narrow-phase collision checks, filtering, and contact manifold generation.

## Features
- **Narrow-phase Dispatch**: Automatically chooses the correct `geometry` test for `Box` (AABB) and `Ball` (Sphere) volumes.
- **Collision Filtering**: Bitmask-based `layer` and `mask` system to ignore irrelevant collisions.
- **Contact Data**: Returns depth, point, and normals for physical resolution.
- **Spatial Integration**: Designed to work seamlessly with `SpatialGrid` for $O(n \log n)$ performance.


## Examples

See [examples.md](examples.md) for usage examples.
## Integration with carp-spatial
The `CollisionChecker.query-and-check` function takes a `SpatialGrid` and a list of entities to perform efficient broad-phase filtered queries.

## License
MIT


---

## Spatial

A high-performance static 3D spatial partitioning grid for the [Carp](https://github.com/carp-lang/Carp) programming language.

## Features

- **Static 3D Grid**: Optimized for fixed-volume simulation spaces.
- **Fast Queries**: Support for AABB, Sphere, and Ray-based spatial queries.
- **Raycasting (DDA)**: Efficiently walks the grid along a ray path to find relevant objects.
- **Zero-Allocation Insertion**: Designed for high-frequency updates in game loops.
- **Robustness**: Handles rays starting outside the grid and precision edge cases.


## Examples

See [examples.md](examples.md) for usage examples.
## Testing

Run the test suite with:

```bash
carp -x test/grid_test.carp
```

## License

MIT


---

## Rigidbody

A high-level physical object unification library for the [Carp](https://github.com/carp-lang/Carp) programming language.

This library unifies **`Transform`** (where an object is) and **`dynamics.carp`** (how an object moves) into a single, cohesive **`RigidBody`** component. It eliminates the boilerplate of passing around pairs of structs and provides a unified API for the entire physics pipeline.

## Features
- **Unified Data**: Combines Position, Rotation, Scale, Velocity, and Force into one type.
- **Clean Proxy API**: Apply forces, impulses, and step simulations directly on the `RigidBody`.
- **Hardened Invariants**: Leverages the stability and safety of the underlying modular physics stack.
- **Engine Ready**: Designed for use in ECS systems, game scaffolds, and solvers.

## Installation
```clojure
(load "git@github.com:carpentry-org/carp-physics-3d@master" "rigidbody.carp")
```


## Examples

See [examples.md](examples.md) for usage examples.
## License
MIT


---

## Physics

A modular, high-performance 3D impulse solver for the [Carp](https://github.com/carp-lang/Carp) programming language.

This library is the "Verb of Interaction" in the modular physics stack. It takes collision data and dynamic states to calculate the impulses and positional corrections needed to resolve physical impacts.

## Features
- **Impulse-Based Resolution**: Textbook implementation of the Sequential Impulse (SI) method for rigid body collisions.
- **Coulomb Friction Model**: Realistic lateral resistance with correct clamping ($|j_t| \le j \cdot \mu$).
- **Baumgarte Stabilization**: Built-in positional correction with configurable slop to prevent "sinking" and jitter.
- **Library Agnostic**: Orchestrates data between `Transform`, `dynamics.carp` and `collision.carp` without tight coupling.
- **Stability First**: Built-in guards for division-by-zero (Static vs Static) and NaN protection for tangent math.

## Installation
```clojure
(load "git@github.com:carpentry-org/carp-physics-3d@master")
```


## Examples

See [examples.md](examples.md) for usage examples.
## The Modular Stack
The solver is the consumer of the modules below it:
1. **`Transform`** (from [carp-math](https://github.com/carpentry-org/carp-math)): authoritative spatial state.
2. **`dynamics.carp`**: newtonian integration and damping.
3. **`collision.carp`**: narrow-phase manifold generation.

## License
MIT


---

## Dynamics

A high-performance Newtonian dynamics and integration library for the [Carp](https://github.com/carp-lang/Carp) programming language.

This library provides the "nouns of motion"—managing forces, velocity, and integration logic. It acts as the bridge between static transforms and a fully simulated physical world.

## Features
- **Physically Consistent Damping**: Uses a time-step safe exponential decay model ($v = v \cdot damping^{dt}$) to prevent numerical energy growth.
- **Semi-Implicit Euler**: Uses the standard game industry integration order (Update Velocity $\to$ Update Position) for maximum stability.
- **Simulation Guardrails**: Built-in Delta-Time clamping to prevent simulation "explosions" during frame spikes or window dragging.
- **Static Body Support**: First-class support for immovable objects with enforced invariants (infinite mass, zero inverse-mass).
- **Lightweight**: Zero external dependencies beyond the standard library and `Transform` from carp-math.

## Installation
```clojure
(load "git@github.com:carpentry-org/carp-physics-3d@master" "dynamics.carp")
```


## Examples

See [examples.md](examples.md) for usage examples.
## Design Philosophy
`dynamics.carp` follows the **Separation of Mechanisms** principle. It handles *how* objects move through space based on forces, but it does not have opinions about *why* they collide. This makes it perfectly suitable for both traditional rigid-body physics and custom SDF-based resolution systems.

## License
MIT


## License

MIT
