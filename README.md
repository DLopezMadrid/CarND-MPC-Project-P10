# CarND-Controls-MPC
Self-Driving Car Engineer Nanodegree Program

[MAIN]: ./imgs/main_pic.png
[EQS]: ./imgs/kin_model_eqs.png


[![IMAGE ALT TEXT](./imgs/main_pic.png)](https://youtu.be/gYLFAwzJcTQ "MPC video")
**Click on the image to go to the youtube video**



---

## Reflection

#### Introduction

In this project, we learned how to use a model predictive controller to steer the vehicle around the circuit. This approach has multiple advantages over using a simple PID controller, for example, instead of being a 'reactive' controller, in this case, the controller is able to look ahead in the future (time horizon) to see if a curve is coming and calculate a more adequate way of taking it. Other examples are the inclusion of a kinematic model that describes the car and the constraints that it has (non-holonomic vehicle).

#### The Model
The kinematic model used in this project is based on a bicycle. Because the model is not dynamic, it neglets all the forces acting on the car such as road and tyre friction and inertias.

When the vehicle is moving a low speeds, the kinematic model are pretty accurate, but at higher speeds they are unable to represent the system properly due to the increased importance of the dynamics.

This model uses the previous timestep to calculate the current state of the vehicle

The model used is described on these equations:
![EQS]

Where :
 - `x` is the longitudinal position of the vehicle
 - `y` is the lateral position of the vehicle
 - `Ψ` (psi) is the heading direction of the vehicle
 - `v` is the velocity of the car
 - `Lf` is the distance beween the front of the vehicle and its center of gravity
 - `𝛿` is the steering angle in radians (actuator)
 - `a` is the acceleration (actuator)
 - `dt` is the time step




#### Timestep Length and Elapsed Duration (N & dt)

The time step lenght and elapsed duration define the time horizon and its granularity. The shorter the time horizon the more responsive (but also more aggresive) the controller is. Longer time horizons generate a smoother behaviour but may fall short when quick actions are required. In short, it is a tradeoff between reacting to the current curvature of the road and looking ahead to plan an ideal path. Having a longer time horizon will also be computationally more expensive.

Because of this (and partly due to the indications given during Udacity office hours ) I chose the following values
`N = 10`
`dt = 0.1`

They work pretty well up to medium speeds (~40mph) but if the speed is increased we need to increase the value of `N` too (too be able to plan better)


#### Polynomial Fitting and MPC Preprocessing

In order to do a polynomial fitting we need to transform the waypoints into the vehicle coordinate system. Once I did that, the origin of the coordinate system coincides with the car and makes easier the process of curve fitting. Finally, I fitted a third order polynomial into the waypoints positions.


#### Model Predictive Control with Latency

Latency presents an issue since the controller will be working continously with "old" data, this can cause the controller to oscillate and erratic vehicle behaviour.
In order to account for the 100ms latency, I changed the equations as it can be seen in MPC.cpp (lines 116 - 120).


---

## Dependencies

* cmake >= 3.5
 * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools]((https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)
* [uWebSockets](https://github.com/uWebSockets/uWebSockets)
  * Run either `install-mac.sh` or `install-ubuntu.sh`.
  * If you install from source, checkout to commit `e94b6e1`, i.e.
    ```
    git clone https://github.com/uWebSockets/uWebSockets
    cd uWebSockets
    git checkout e94b6e1
    ```
    Some function signatures have changed in v0.14.x. See [this PR](https://github.com/udacity/CarND-MPC-Project/pull/3) for more details.
* Fortran Compiler
  * Mac: `brew install gcc` (might not be required)
  * Linux: `sudo apt-get install gfortran`. Additionall you have also have to install gcc and g++, `sudo apt-get install gcc g++`. Look in [this Dockerfile](https://github.com/udacity/CarND-MPC-Quizzes/blob/master/Dockerfile) for more info.
* [Ipopt](https://projects.coin-or.org/Ipopt)
  * Mac: `brew install ipopt`
       +  Some Mac users have experienced the following error:
       ```
       Listening to port 4567
       Connected!!!
       mpc(4561,0x7ffff1eed3c0) malloc: *** error for object 0x7f911e007600: incorrect checksum for freed object
       - object was probably modified after being freed.
       *** set a breakpoint in malloc_error_break to debug
       ```
       This error has been resolved by updrading ipopt with
       ```brew upgrade ipopt --with-openblas```
       per this [forum post](https://discussions.udacity.com/t/incorrect-checksum-for-freed-object/313433/19).
  * Linux
    * You will need a version of Ipopt 3.12.1 or higher. The version available through `apt-get` is 3.11.x. If you can get that version to work great but if not there's a script `install_ipopt.sh` that will install Ipopt. You just need to download the source from the Ipopt [releases page](https://www.coin-or.org/download/source/Ipopt/) or the [Github releases](https://github.com/coin-or/Ipopt/releases) page.
    * Then call `install_ipopt.sh` with the source directory as the first argument, ex: `sudo bash install_ipopt.sh Ipopt-3.12.1`.
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [CppAD](https://www.coin-or.org/CppAD/)
  * Mac: `brew install cppad`
  * Linux `sudo apt-get install cppad` or equivalent.
  * Windows: TODO. If you can use the Linux subsystem and follow the Linux instructions.
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page). This is already part of the repo so you shouldn't have to worry about it.
* Simulator. You can download these from the [releases tab](https://github.com/udacity/self-driving-car-sim/releases).
* Not a dependency but read the [DATA.md](./DATA.md) for a description of the data sent back from the simulator.


## Basic Build Instructions


1. Clone this repo.
2. Make a build directory: `mkdir build && cd build`
3. Compile: `cmake .. && make`
4. Run it: `./mpc`.

## Tips

1. It's recommended to test the MPC on basic examples to see if your implementation behaves as desired. One possible example
is the vehicle starting offset of a straight line (reference). If the MPC implementation is correct, after some number of timesteps
(not too many) it should find and track the reference line.
2. The `lake_track_waypoints.csv` file has the waypoints of the lake track. You could use this to fit polynomials and points and see of how well your model tracks curve. NOTE: This file might be not completely in sync with the simulator so your solution should NOT depend on it.
3. For visualization this C++ [matplotlib wrapper](https://github.com/lava/matplotlib-cpp) could be helpful.

## Editor Settings

We've purposefully kept editor configuration files out of this repo in order to
keep it as simple and environment agnostic as possible. However, we recommend
using the following settings:

* indent using spaces
* set tab width to 2 spaces (keeps the matrices in source code aligned)

## Code Style

Please (do your best to) stick to [Google's C++ style guide](https://google.github.io/styleguide/cppguide.html).

## Project Instructions and Rubric

Note: regardless of the changes you make, your project must be buildable using
cmake and make!

More information is only accessible by people who are already enrolled in Term 2
of CarND. If you are enrolled, see [the project page](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/f1820894-8322-4bb3-81aa-b26b3c6dcbaf/lessons/b1ff3be0-c904-438e-aad3-2b5379f0e0c3/concepts/1a2255a0-e23c-44cf-8d41-39b8a3c8264a)
for instructions and the project rubric.

## Hints!

* You don't have to follow this directory structure, but if you do, your work
  will span all of the .cpp files here. Keep an eye out for TODOs.

## Call for IDE Profiles Pull Requests

Help your fellow students!

We decided to create Makefiles with cmake to keep this project as platform
agnostic as possible. Similarly, we omitted IDE profiles in order to we ensure
that students don't feel pressured to use one IDE or another.

However! I'd love to help people get up and running with their IDEs of choice.
If you've created a profile for an IDE that you think other students would
appreciate, we'd love to have you add the requisite profile files and
instructions to ide_profiles/. For example if you wanted to add a VS Code
profile, you'd add:

* /ide_profiles/vscode/.vscode
* /ide_profiles/vscode/README.md

The README should explain what the profile does, how to take advantage of it,
and how to install it.

Frankly, I've never been involved in a project with multiple IDE profiles
before. I believe the best way to handle this would be to keep them out of the
repo root to avoid clutter. My expectation is that most profiles will include
instructions to copy files to a new location to get picked up by the IDE, but
that's just a guess.

One last note here: regardless of the IDE used, every submitted project must
still be compilable with cmake and make./
