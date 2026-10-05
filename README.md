# TurtleBot3 Localization & Navigation (EKF + DWA)

A small mobile robot (a **TurtleBot3**) that works out where it is, then follows a moving target while steering around obstacles. This repository holds the code, the two written reports, and a video of the results.


---

## The short version

For a robot to get around on its own, it has to keep answering two questions, many times per second:

1. **"Where am I right now?"** Answered here with an **Extended Kalman Filter (EKF)**.
2. **"Which way should I move next, without hitting anything?"** Answered here with the **Dynamic Window Approach (DWA)**.

The first part of the project builds the EKF. The second part builds the DWA, using the robot's LiDAR (a spinning laser distance sensor) to see obstacles and its camera to follow a marker (an AprilTag).

---

## What's in this repository

| File | What it is |
|---|---|
| `video_robot_sesasr.mp4` | **Start here.** A video of the robot in action. |
| `103_Report02.pdf` | **Report 2: the EKF.** How the robot estimates its position from wheel odometry, an IMU and landmarks seen by the camera. Includes simulation and real-robot results. |
| `103_Report03.pdf` | **Report 3: the DWA.** How the robot follows a moving target and avoids obstacles. Includes simulation and real-robot results. |
| `Dynamic Window Approach.zip` | Source code (zipped). [Add what's inside, for example the ROS 2 package for the DWA node and the EKF node] |
| `LICENSE` | GPL-3.0 open-source license. |


---

## Part 1: Where am I? (Extended Kalman Filter)

**The problem.** A robot can't just *know* its position. Wheel counters drift as the wheels slip, and every sensor is a little noisy.

**The idea.** An EKF combines two imperfect sources of information into one better guess:

- **Predict:** "I was here a moment ago and I've been driving at this speed, so I should be about *there*."
- **Correct:** "My camera just spotted a landmark whose position I know, so I must be about *here*." The filter weighs how much to trust each one and blends them.


**What we did** (see `103_Report02.pdf`):
- Wrote the EKF as a **ROS 2 node** that predicts 20 times per second and corrects whenever a landmark is seen.
- Tested it in **simulation** against the simulator's true position, and on the **real TurtleBot3** using landmarks seen by its onboard camera.
- Extended the filter to also use the **IMU** (a motion sensor) and wheel encoders, a technique called *sensor fusion*.

**What we found**
- In simulation, the EKF's position error was roughly **3 times smaller** than raw odometry (about 0.004 m compared with about 0.011 m).
- On the real robot there is no ground truth to compare against, but the EKF path was smoother and handled tight turns better than odometry alone.

---

## Part 2: Where do I go next? (Dynamic Window Approach)

**The problem.** The robot needs to chase a moving target and not crash into anything on the way.

**The idea.** Many times per second, DWA does this:

1. **Considers many options:** combinations of "how fast forward" and "how much to turn" that the robot can reach right now (it can't change speed instantly).
2. **Imagines each one:** "If I did this for the next fraction of a second, where would I end up?"
3. **Scores them:** options that point toward the target, keep a good speed, and stay clear of obstacles score best.
4. **Picks the best one,** drives that way briefly, then repeats.

Because it re-plans all the time, the robot can react when an obstacle appears or the target moves.

**What we did** (see `103_Report03.pdf`):
- Built a ROS 2 node that runs the loop 15 times per second, reading the LiDAR for obstacles.
- Added **safety layers**: unsafe options are thrown out, and an emergency stop triggers if something gets too close.
- Improved the scoring so the robot **slows down near the target** and **keeps a steady following distance**.
- Moved it from simulation to the **real robot**, where the target is an **AprilTag** held in front of the camera. This meant cleaning up noisy LiDAR readings and re-tuning the safety limits for a small lab.

**What we found**
- **Simulation:** the robot followed a moving target along a winding path with no close calls to obstacles, staying within 2 m of the target the whole time.
- **Real robot (3 runs):** the robot kept tracking the tag in every run. The best run had no close calls. In the hardest run there were some close calls (the nearest obstacle was about 0.15 m away).

---

## Tools used

- **Robot:** TurtleBot3 (LiDAR, camera, IMU, wheel encoders)
- **Software:** ROS 2, Gazebo (simulation), Python, SymPy (for the math), PlotJuggler and rosbag (recording and plotting data)
- **Methods:** Extended Kalman Filter, sensor fusion, Dynamic Window Approach

---


## Credits

A team project by Group 103: Mehmood Sarkhel, Gianluca Lamparelli, Riccardo Ferranti and Riccardo Della Malva.

## License

Released under the GPL-3.0 license. See `LICENSE` for details.
