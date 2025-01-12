# Phase 1: Basics

[gimmick: math]()

This is the first phase of the semester-long project. It serves as a good litmus test--if you struggle implementing it once you understand the problem, you're likely to flounder in the rest of the class.

The next few sections explain what the code is doing; if you don't care, feel free to skip to the [pseudocode](#pseudocode) and [assignment](#the-assignment).




## The problem

Your job is to simulate a simple [damped](https://en.wikipedia.org/wiki/Damping) version of the [wave equation](https://en.wikipedia.org/wiki/Wave_equation):

$$\ddot u = \nabla^2 u - c \space \dot u$$

...where \\(c\\) is the damping coefficient, \\(u\\) represents displacement from equilibrium, \\(\dot u\\) represents the first derivative of \\(u\\) with respect to time (and \\(\ddot u\\) the second), and \\(\nabla^2\\) is the [Laplace operator](https://en.wikipedia.org/wiki/Laplace_operator).



### Evolution in time

To be clear, you don't have to know anything about [calculus](https://www.3blue1brown.com/lessons/essence-of-calculus) or [differential equations](https://www.3blue1brown.com/lessons/differential-equations) for this course (although a basic understanding will enrich your understanding of the world). All you need to know is that this simplified wave equation can be discretized with a [leapfrog method](https://en.wikipedia.org/wiki/Leapfrog_integration) as:

$$v_I^{(t+dt)} = (1-dt \space c) v_I^{(t)} + dt \space \nabla^2 u_I^{(t)}$$


$$u_I^{(t+dt)} = u_I^{(t)} + dt \space v_I^{(t+dt)}$$

...where \\(c\\) is again the damping coefficient, \\(u\\) is the displacement, \\(v\\) is its velocity, \\(I\\) is an index in the discretized grid, \\(t\\) is the simulation time, and \\(dt\\) is the time step.

We [approximate the Laplacian](https://en.wikipedia.org/wiki/Discrete_Laplace_operator#Finite_differences) as:

$$\nabla^2 x_{I} \approx \left( \sum_n^{2N} \frac{x_{n}}{2} \right) - N \space x$$

...where \\(N\\) is the dimension of the problem and each of \\(n\\) are indices adjacent to \\(I\\) in positive and negative directions along each axis; in two dimensions this is:

$$\nabla^2 x_{i,j} \approx \frac{x_{i,j-1}+x_{i,j+1}+x_{i-1,j}+x_{i+1,j}}{2} - 2 \space x_{i,j}$$

...for cell \\((i, j)\\).



### Energy

To determine the energy stored in a 2-dimensional rectangular wave plane, we imagine each cell as a particle of unit mass, each connected to the four adjacent particles by springs of unit spring coefficient constrained to move along the axis orthogonal to the plane.

Since each particle is of unit mass, the kinetic energy of a cell is:

$$KE_{I}^{(t)} = \frac{\left(v_{I}^{(t)}\right)^2}{2}$$

The potential energy is held not in cells, but in the boundaries between cells (the springs in our example case). Given unit spring coefficient, the potential energy between two adjacent cells is:

$$PE_{I;J}^{(t)} = \frac{\left( u_{I}^{(t)} - u_{J}^{(t)} \right)^2}{4}$$

Division is by 4 rather than two because both ends of each spring are free; it can thus be modeled as a central point with springs of half length on each side, with half the potential energy of a system with only one free end. The potential of the boundaries at the edges is made identical (and, importantly, the simulation is simplified) by considering these "springs" to have half the spring coefficient of interior "springs."





## The simulation

Given a damping coefficient of 0.01, a time step of 0.01, an initial grid of size 25x50 with [edges fixed at zero](https://en.wikipedia.org/wiki/Dirichlet_boundary_condition), displacement $u$ initialized to all zeros, and the interior of displacement velocity $v$ initialized to 0.1:

$$u_0 = \begin{bmatrix}
    0      & \dots  & 0      \\
    \vdots & \ddots & \vdots \\
    0      & \dots  & 0      \\
\end{bmatrix}$$

$$v_0 = \begin{bmatrix}
    0      & 0      & \dots  & 0      & 0      \\
    0      & 0.1    & \dots  & 0.1    & 0      \\
    \vdots & \vdots & \ddots & \vdots & \vdots \\
    0      & 0.1    & \dots  & 0.1    & 0      \\
    0      & 0      & \dots  & 0      & 0      \\
\end{bmatrix}$$

...your job is to determine how much time it takes for the average energy of cells in the interior grid to fall below 0.001--that is, how much time it takes for the total system energy to drop below $\left(25-2\right) \times \left(50-2\right) / 1000$.



### Visualization

To make this a bit more concrete, here's what the displacement looks like through the course of the simulation (visualized as an elastic membrane):

![two-d-sim](../uploads/phase-1-animation-2D.gif)

In one dimension, it could look something like this (visualized as an elastic string):

![one-d-sim](../uploads/phase-1-animation-1D.gif)

As you can see, once the initial conditions are set, each cell simply seeks the average of the surrounding cells given the constraints imposed by momentum and the fixed boundary conditions.





## Pseudocode

### Conventions

- arrays are [indexed starting at 1](https://medium.com/analytics-vidhya/array-indexing-0-based-or-1-based-dd89d631d11c); be careful when translating to C++, which indexes from 0.
- `zeros(4, 5)` returns a 4x5 array of floating point zeros.
- `for i in 2:4` iterates from 2 to 4 inclusive.
- floating point numbers are of double precision.



### The Algorithm

```c++
// Initialize
dt = 0.01
m = 25
n = 50
c = 0.01
t = 0
u = zeros(m, n)
v = zeros(m, n)
v[2:m-1,2:n-1] = 0.1

energy = m * n
while energy > (m - 2) * (n - 2) / 1000 {
    // Update v
    for i in 2:m-1, j in 2:n-1 {
        L = (u[i-1,j] + u[i+1,j] + u[i,j-1] + u[i,j+1]) / 2 - 2 * u[i,j]
        v[i,j] = (1 - dt * c) * v[i,j] + dt * L
    }
    // Update u
    for i in 2:m-1, j in 2:n-1 {
        u[i,j] += dt * v[i,j]
    }
    energy = 0
    // Add kinetic energy
    for i in 2:m-1, j in 2:n-1 {
        energy += v[i,j]^2 / 2
    }
    // Add potential energy along first axis (note i range)
    for i in 1:m-1, j in 2:n-1 {
        energy += (u[i,j] - u[i+1,j])^2 / 4;
    }
    // Add potential energy along second axis (note j range)
    for i in 2:m-1, j in 1:n-1 {
        energy += (u[i,j] - u[i,j+1])^2 / 4;
    }
    t += dt
}

print(t)
```





## The Assignment

Your job is to (loosely) translate the above pseudocode to working C++20 code. You'll create a C++20 file, and optionally associated headers, that can be compiled with G++ 12 using the following invocation:

```shell
g++ -std=c++20 -o wave_solve wave_solve.cpp
```

When run, the resulting binary should print "155.77", followed by a newline, to stdout and return zero. I don't care about the correct answer as much as the means of getting to it.



### Resources

Since many students aren't familiar with modern C++, I've created a [skeleton](#appendix-a-skeleton-code) that you can use as a starting point. There are 4 sections that need to be implemented, marked with "`# TODO`" comments. If you haven't used C++ extensively, I would strongly recommend using the skeleton code as your foundation (even if doing so is harder than coding the way you're used to!) and checking out [the W3Schools C++ tutorial](https://www.w3schools.com/cpp/).

If you don't have access to and practice with a C++20 compiler for testing, an online C++ compiler (such as [OnlineGDB](https://www.onlinegdb.com/online_c++_compiler)) will work. Usually you'll have to find a menu that allows you to use C++20; in the case of OnlineGDB, it's the dropdown at the upper right that says "C++".





## Appendix A: Skeleton Code

```c++
#include <array>
#include <vector>
#include <iostream>



// Class representing a rectangular plane with Dirichlet boundary conditions over which waves propagate
// Called "orthotope" rather than "rectangle" since extra credit is given later for generalizing to arbitrary dimension
class WaveOrthotope {
    // Types
    using size_type = size_t;
    using value_type = double;

    // Members
    const size_type m, n;              // size
    const value_type c;                // damping coefficient
    value_type t;                      // simulation time
    std::vector<value_type> disp, vel; // displacement and velocity arrays

public:
    // Constructor
    // USAGE: auto my_wave_orthotope = WaveOrthotope(20, 30, 0.1, 0);
    // SEE: https://www.learncpp.com/cpp-tutorial/constructor-member-initializer-lists/
    WaveOrthotope(const auto m, const auto n, const auto c, const value_type t=0): m(m), n(n), c(c), t(t), disp(m*n), vel(m*n) {}

    // Return the size as a std::array
    // USAGE: const auto [rows, cols] = my_wave_orthotope.size();
    // SEE: https://codeburst.io/c-17-structural-binding-180696f7a678#63b1
    constexpr const auto size() const { return std::array{m, n}; }

    // Displacement indexing
    // USAGE: my_wave_orthotope.u(1, 2) = 1.0;
    constexpr const value_type  u(const auto i, const auto j) const { return disp[i*n+j]; }
    constexpr       value_type& u(const auto i, const auto j)       { return disp[i*n+j]; }

    // Displacement velocity indexing
    // USAGE: const auto v12 = my_wave_orthotope.v(1, 2);
    constexpr const value_type  v(const auto i, const auto j) const { return vel [i*n+j]; }
    constexpr       value_type& v(const auto i, const auto j)       { return vel [i*n+j]; }

    // Return the energy contained in this WaveOrthotope
    // USAGE: const auto E = my_wave_orthotope.energy();
    constexpr const value_type energy() const {
        T E{};
        // TODO: calculate total energy
        return E;
    }

    // Advance the membrane in time by dt
    // USAGE: my_wave_orthotope.step();
    constexpr const value_type step() {
        const T dt = 0.01;
        // TODO: update u and v by one time step
        t += dt;
        return t;
    }

    // Advance the membrane in time by steps of dt until the average interior cell energy drops below 0.001
    // USAGE: const auto sim_time = my_wave_orthotope.solve();
    constexpr const value_type solve() {
        while (energy() > (m-2)*(n-2)*0.001) {
            step();
        }
        return t;
    }
};



int main() {
    const size_t m = 25, n = 50;
    const double c = 0.01, t = 0, interior_velocity = 0.1;
    // TODO: create a WaveOrthotope given m, n, c, and t and fill the interior of its velocity with interior_velocity
    // TODO: solve the WaveOrthotope then print out its simulation time
    return 0;
}
```