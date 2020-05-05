---
layout: post
title: Basic ROS2 GPS Node (still a work in progress)
date: '2020-05-05 11:30'
---

# This is a basic ROS2 Node using the C++ ROS Client Library
- The code can be found [here](#code).
- The CMakeList file can be found [here](#cmakelist).
- Useful Links for this project can be found [here](#links)
---

> The purpose is to connect to a GPS sensor (which is connected to a RS232-USB converter that is connected to the /dev/ttyUSB0 port)
> I was just trying to use the [serial driver](https://github.com/RoverRobotics-forks/serial-ros2) provided by WJWood.
> It took a little _Friggin_ around with the CMakeList.txt file to get it to compile properly. I will add the contents of that file later


# Code
```cpp class:"lineNo"
#include <string>
#include <iostream>
#include <vector>
#include <sstream>
#include <cstdio>

#include <chrono>
#include <memory>

#include "rclcpp/rclcpp.hpp"
#include "std_msgs/msg/string.hpp"
#include "sensor_msgs/msg/nav_sat_fix.hpp"
#include "sensor_msgs/msg/nav_sat_status.hpp"


#include "../include/serial/serial.h"

using namespace std::chrono_literals;

/* This example creates a subclass of Node and uses std::bind() to register a
 * member function as a callback from the timer. */

class serialGPS : public rclcpp::Node
{

  //initialize all class members
  std::unique_ptr<sensor_msgs::msg::NavSatFix> fix_ = nullptr;

  std_msgs::msg::Header header_;
  rclcpp::TimerBase::SharedPtr timer_;
  rclcpp::Publisher<std_msgs::msg::String>::SharedPtr publisher_;
  rclcpp::Publisher<sensor_msgs::msg::NavSatFix>::SharedPtr nav_pub_;

  serial::Serial serial_;
  bool pubNavSat;


public:
  serialGPS()
  //serialGPS Constructor. Initializes the base Node object, serial_port object
  : Node("serialGPS"), pubNavSat(false), serial_("/dev/ttyUSB0", 4800, serial::Timeout::simpleTimeout(1000))
  {

    rmw_qos_profile_t navsat_profile  = rmw_qos_profile_default;
    rmw_qos_profile_t gps_profile     = rmw_qos_profile_sensor_data;

    auto navqos = rclcpp::QoS(
                    rclcpp::QoSInitialization(
                      navsat_profile.history,
                      navsat_profile.depth
                    ),
                    navsat_profile
                  );
    auto gpsqos = rclcpp::QoS(
                    rclcpp::QoSInitialization(
                      gps_profile.history,
                      gps_profile.depth
                      ),
                  gps_profile);


    //rclcpp::QoS qos(rclcpp::KeepLast(7));

    publisher_ = this->create_publisher<std_msgs::msg::String>("GPS_Data", gpsqos);
    nav_pub_ = this->create_publisher<sensor_msgs::msg::NavSatFix>("NavSatFix", navqos);

    //should this be initialised with the constructor?


    timer_ = this->create_wall_timer(
      500ms, std::bind(&serialGPS::timer_callback, this));

    RCLCPP_INFO(this->get_logger(), "Is the serial port open?");
    if(serial_.isOpen())
      RCLCPP_INFO(get_logger(), "Yes.");
    else
      RCLCPP_INFO(get_logger(), "No.");

      //std::cout << "Is the serialport open? " << serial_.isOpen()? "Yes\n" : "No\n";

  }

private:

  void timer_callback()
  {
    std::cout << "Callback " << std::endl;
    fix_ = std::make_unique<sensor_msgs::msg::NavSatFix>();

    auto message = std_msgs::msg::String();

    std::string gpsMessage = "";
    std::vector<std::string> gpsData;
    processGPS(gpsMessage, gpsData);

    message.data = gpsMessage;
    //if(message.data != "")
    //{
      RCLCPP_INFO(this->get_logger(), "Publishing: '%s'", message.data.c_str());
      publisher_->publish(std::move(message));
    //}
    if(pubNavSat)
    {
      RCLCPP_INFO(this->get_logger(), "Publishing: NavSatFix Message");
      nav_pub_->publish(std::move(fix_));
      pubNavSat = false;
    }
  }



  void processGPS(std::string &gpsMsg, std::vector<std::string> &gpsData)
  {

    gpsMsg = serial_.readline();

    std::stringstream ss(gpsMsg);
    //std::vector<std::string> gpsData;

    while( ss.good() )
    {
      std::string substr;
      std::getline( ss, substr, ',');
      gpsData.push_back( substr );
    }

    if(gpsData[0] == "$GPGGA")
    {
      double temp;
      try
      {
        fix_->latitude = std::stod(gpsData[2]);
        temp = std::stod(gpsData[4]);
      }catch(std::exception e) {std::cout << e.what();}

      if(gpsData[5] == "W")
        temp = temp *(-1);
      fix_->longitude = temp;
    }
    else
    {
      fix_->latitude = 2;
      fix_->longitude = -6;
    }

    fix_->header.stamp = this->get_clock()->now();
    fix_->header.frame_id = "serial gps";

    fix_->status.status = 0;
    fix_->status.service = 1;


    fix_->altitude = 0;
    fix_->position_covariance = {0,0,0,0,0,0,0,0,0};
    fix_->position_covariance_type = 0;
    pubNavSat = true;


  }

};




int main(int argc, char * argv[])
{
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<serialGPS>());
  rclcpp::shutdown();
  return 0;
}

```
---
# CMakeList
> Here is the CMakeList.txt File. I will comment the changes below

```CMakeList

cmake_minimum_required(VERSION 3.5)
project(serial)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})

## Sources
## Add serial library
add_library(${PROJECT_NAME}
    src/serial.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)


target_sources(${PROJECT_NAME} PRIVATE
    src/impl/unix.cc
    src/impl/list_ports/list_ports_linux.cc
)

## Include headers
target_include_directories(${PROJECT_NAME} PRIVATE include)

## Uncomment for example
 add_executable(serial_gps examples/serialGPS.cc)
 ament_target_dependencies(serial_gps rclcpp std_msgs sensor_msgs)
 add_dependencies(serial_gps ${PROJECT_NAME} )

 target_link_libraries(serial_gps ${PROJECT_NAME})



## Install executable
install(TARGETS ${PROJECT_NAME}
    serial_gps
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    DESTINATION lib/${PROJECT_NAME}
)

## Install headers
install(FILES include/serial/serial.h include/serial/v8stdint.h
    DESTINATION include/serial
)

## Tests
#if(BUILD_TESTING)
#    add_subdirectory(tests)
#endif()

ament_package()

```

> With regards to the CMakeList.txt file, I needed to change a few things;


 - > The following is used for local dependancies (ie, header files that are part of the package)
   > It is relative to the package path (~/home/username/ros2_workspace/src/). It is relative to where the package CMakeList.txt file resides



        ## Sources
        ## Add serial library
        add_library(${PROJECT_NAME}
            src/serial.cc
            include/serial/serial.h
            include/serial/v8stdint.h
        )

- > Under ##Uncomment for examples, you must add the following line for colcon to build

      ament_target_dependencies(serial_gps rclcpp std_msgs sensor_msgs)

- > In the ##Install Executable subsection, you must include:

      DESTINATION lib/${PROJECT_NAME}

- > Or the *ros2 run* command will not identify the executable within the find_package


- > To run colcon on only the serial_gps package, run:

      colcon build --packages-select serial_gps --symlink-install

- > And Don't forget to source your workspace after successful compilation

      . /home/<username>/<ROS2_workspace>/install/setup.bash
      
      
# Links
- a ROS1 [gps package](https://github.com/swri-robotics/gps_umd/blob/dashing-devel/gpsd_client/src/client.cpp)
