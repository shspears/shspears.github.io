---
layout: "post"
title: "Colcon Notes"
date: "2020-05-05 19:08"
---

# Colcon information

> Colcon is the build tool for ROS2. It combines ament and cmake, to make your compiler (if you're using Linux, it's most likely gcc) aware of any ROS2 dependencies in order to build (compile / link) any ROS2 Code contained within a ROS2 package.

> In order to use colcon to build packages,
>  1. your workspace must be set up properly. See [here](https://shspears.github.io/posts/ROS2/) to install ROS2 and configure the environment.
> 2. The given package you're trying to build must have a properly configured  [CMakeList.txt](#cmakelist) and [package.xml](#package) files

---
## Colcon Cheat Sheet
![Cheet Sheet](https://shspears.github.io/images/colcon_cheatsheet_1.png "Colcon Cheet Sheet")
---


## Useful commands

- Build only the selected <package-name> package
```
colcon build --packages-select <package_name> --symlink-install
```



> Sometimes Colcon uses cached CMakeCache.txt files and does not properly build a given package. Removing that packages' folder from /home/<username>/<ros2_ws>/build/ directory will help in being able to recompile
