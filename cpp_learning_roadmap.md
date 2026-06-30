# C++ Learning Roadmap: From Fundamentals to Scientific Software

A self-directed curriculum structured around two physics projects (heat diffusion, then gravity anomaly modelling) and culminating in reading professional FEM codebases, with MFEM as the target.

Time budget assumed: a few hours a week, alongside job hunting and other commitments. Treat the phase lengths as guides, not deadlines — the point of each phase is the checkpoint, not the calendar.

---

## Phase 0: Setup and habits

Before starting, get the scaffolding right so friction doesn't slow you down later.

- Set up CMake as your build system from day one, even for trivial programs. Learning to write a `CMakeLists.txt` for a two-file program now saves pain later when your FEM project has multiple translation units.
- Pick a compiler and stick with it (GCC or Clang). Enable warnings aggressively (`-Wall -Wextra -Wpedantic`) from the start — this catches a huge fraction of beginner mistakes early.
- Set up a debugger (gdb or lldb) and learn to use it for at least basic breakpoint/step/inspect workflows. Don't rely on print statements as your only debugging tool.
- Use git from the first exercise. Your eventual project repos are also useful evidence for job applications.

---

## Phase 1: Language fundamentals

**Primary text:** *Discovering Modern C++* (Gottschling)

**Goal:** Be fluent enough in modern C++ idioms that you stop writing C-with-classes by default.

### Core topics to master (not just read)

Work through these roughly in order. For each, the test of understanding is whether you can explain *why* the feature exists, not just what it does.

1. **Basic syntax and types** — value vs reference semantics, `const` correctness, references vs pointers.
2. **Functions and overloading** — function templates as an early taste of generic programming.
3. **Classes and object lifetime** — constructors, destructors, the rule of zero/three/five.
4. **RAII** — this is the single most important C++ idiom. Understand it deeply before moving on. Everything else (smart pointers, containers, locks) is RAII applied to a specific problem.
5. **Move semantics and rvalue references** — why they exist (avoiding unnecessary copies), and the mechanics of `std::move`, move constructors, move assignment.
6. **Smart pointers** — `unique_ptr`, `shared_ptr`, when to use which, and why raw `new`/`delete` should almost never appear in your code.
7. **Templates** — function templates, class templates, and at least an introduction to template metaprogramming concepts (you don't need to be an expert here yet).
8. **The STL** — containers (`vector`, `array`, `map`), iterators, and the algorithms library (`<algorithm>`). Get comfortable with range-based for loops and lambda expressions.
9. **Error handling** — exceptions vs error codes, when each is appropriate in numerical code (this matters for Phase 2 — solver failures need sensible handling).

### Small exercises (do these as you go, not all at the end)

Keep these tiny — the goal is reps, not ambition:

- Implement a minimal dynamic array class (to understand what `std::vector` does under the hood, and to practise RAII and move semantics).
- Implement a simple `Matrix` class supporting basic operations (`+`, `*`, indexing) — this becomes the seed of your linear algebra code for Phase 2.
- Implement a basic linked list or binary tree (classic data structure practice).
- Write a simple iterative solver (e.g. Jacobi or Gauss-Seidel for a small linear system) using only what you've learned so far.

### Phase 1 checkpoint

You should be able to: write a templated container class with correct move semantics, use `std::vector` and STL algorithms idiomatically, and explain RAII to someone else without notes.

---

## Phase 2A: 1D heat diffusion solver

**Why heat diffusion first:** it naturally extends from the simplest possible problem (steady-state, i.e. just $\nabla^2 T = 0$) into a richer one (time-dependent diffusion), so you always have a working program at each stage and the complexity ramp is gentle. It's also the canonical FEM teaching problem, so reference material is everywhere if you get stuck.

### Stage 1: Steady-state, 1D

Solve

$$-\frac{d^2T}{dx^2} = f(x)$$

on a 1D domain with Dirichlet boundary conditions, using finite differences first (simpler than FEM, good for getting the linear algebra and boundary condition handling right), then re-implement using a basic finite element formulation (linear basis functions on a 1D mesh).

Design decisions to make deliberately (don't just copy a reference implementation without understanding the choice):
- How do you represent the mesh? (Just an array of node positions is fine for 1D.)
- How do you assemble the global stiffness matrix from element contributions?
- How do you impose boundary conditions in the linear system?
- What solver do you use? (Direct solve is fine at this scale — save iterative solvers for Phase 2B.)

### Stage 2: Time-dependent, 1D

Extend to

$$\frac{\partial T}{\partial t} = \kappa \frac{\partial^2 T}{\partial x^2}$$

using an explicit or implicit time-stepping scheme (backward Euler is a reasonable, stable choice to start with). This introduces:
- Time-loop structure and how to manage state across time steps.
- The stability/accuracy tradeoff between explicit and implicit schemes (worth understanding conceptually even if you only implement one).

### Phase 2A checkpoint

A working 1D heat diffusion solver, validated against an analytical solution (e.g. a known steady-state profile, or comparison to the exact solution of the heat equation for simple initial conditions). Code should be organised into separate classes/modules for mesh, assembly, and solver — not one monolithic `main()`.

---

## Phase 2B: 2D gravity anomaly solver

**Why this is the more scientifically motivated project:** this connects directly to your geophysics background and to your longer-term interest in contributing to open-source geophysics tools.

The problem: solve Poisson's equation for the gravitational potential given a density distribution,

$$\nabla^2 \phi = 4\pi G \rho$$

then compute the resulting gravity anomaly at the surface from the potential field.

### Stage 1: Structured grid, simple geometry

Start with a 2D structured (regular) grid and a simple density anomaly (e.g. a buried disk or rectangle of known density contrast in an otherwise uniform background). This has a semi-analytical comparison solution for simple shapes, which gives you a validation target.

New C++ concepts this stage will force you to confront:
- 2D array/matrix storage and indexing conventions (row-major vs column-major, and why it matters for performance).
- Sparse matrix representation — your stiffness matrix is now large and mostly zero; storing it densely is wasteful. This is a natural point to learn about sparse matrix formats (CSR is the standard).
- Iterative solvers — at this scale, direct solvers start to become inefficient. Implement (or use a library for) conjugate gradient, since the system is symmetric positive definite.

### Stage 2: Irregular geometry / unstructured mesh

Extend to an unstructured triangular mesh, allowing more realistic density anomaly shapes. This is a substantial step up in complexity:
- Mesh generation (you can use an existing simple mesh generator/library here rather than writing your own — Triangle or a similar tool is fine; the point is the FEM solver, not mesh generation).
- Element-by-element assembly on triangular elements, including computing element stiffness matrices via numerical quadrature.
- More general boundary condition handling.

### Phase 2B checkpoint

A working 2D gravity anomaly forward solver, with at least one validated test case against an analytical or semi-analytical solution. This is the artefact most worth putting on GitHub for job applications — it demonstrates real scientific software engineering, not toy exercises.

### Optional extension

If you want to bring in your Bayesian inference background: pose the inverse problem (recover density distribution from surface gravity observations) using a simple regularised least-squares or Bayesian approach. This would be a natural full-circle project given your dissertation work, though it's a meaningful additional undertaking — treat it as optional rather than part of the core path.

---

## Phase 3: Reading MFEM (and other professional codebases)

**Start this in parallel with Phase 2B, not after it.** Reading code is a distinct skill from writing it, and starting early means you have your own (much simpler) implementation fresh in mind as a point of comparison.

### Approach

1. Read MFEM's own tutorial examples (`ex1.cpp` through the early-numbered examples) — these are designed to be read in order and build up in complexity, much like your own project.
2. After you've implemented your own 2D Poisson solver, find the MFEM equivalent and compare line by line. Ask: where do they generalise something I hard-coded? Where do they use an abstraction I didn't think to introduce?
3. Pick one subsystem to understand deeply rather than trying to grasp the whole library — a good candidate given your project is MFEM's handling of finite element spaces and assembly, since you'll have direct intuition for what it needs to do.
4. Once comfortable, try solving your gravity anomaly problem *using* MFEM rather than your own code, and compare the experience — this is where David's suggested benefit (learning what professional library design looks like) really lands.

### Other libraries worth a look once comfortable

- **Eigen3** — dense and sparse linear algebra, excellent reference for how to design clean, efficient C++ numerical interfaces (header-only, very readable).
- Your own FMTOMO modernisation interest is a natural longer-term target once you're comfortable reading and writing in this space — the skills from this roadmap transfer directly.

---

## Suggested pacing (loose, adjust freely)

| Phase | Focus | Rough duration at a few hrs/week |
|---|---|---|
| 0 | Setup | A few days |
| 1 | Language fundamentals | 6-10 weeks |
| 2A | 1D heat diffusion | 3-5 weeks |
| 2B | 2D gravity anomaly | 8-14 weeks |
| 3 | Reading MFEM | Ongoing, starting partway through 2B |

Don't be afraid to loop back into Phase 1 material as needed — you'll understand templates much better once you actually need them for Phase 2B's sparse matrix code than you will from reading about them in isolation.
