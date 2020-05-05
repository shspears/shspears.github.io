---
layout: "post"
title: "Ros2 QoS Policy information"
date: "2020-05-05 14:59"
---

# QoS Policy information for ROS2's DDS middleware
---
## This aims to be a half decent guide for QoS policies with DDS on ROS2

- [Policies Source Code](#ros2-qos-policies)
- [Definitions of policies](#some-useful-qos-parameter-definitions)
- [Predefined QoS Policies](#predefined-qos-policies)
- [Examples](#code-examples)


> - Some information that is contained here was taken from [here](https://surfertas.github.io/ros2/2019/08/17/ros2-qos.html).

### ROS2 Middleware Interface API can be found:

- [Middleware API document](http://docs.ros2.org/eloquent/api/rmw/index.html)
- [QoS Policy profiles](https://github.com/ros2/rmw/blob/master/rmw/include/rmw/qos_profiles.h)
- [RMW Source code on Github](https://github.com/ros2/rmw/)

# ROS2 QoS policies
### Policies are defined in a struct. This struct can be found [here](https://github.com/ros2/rmw/blob/master/rmw/include/rmw/types.h)

```cpp
/// ROS MiddleWare quality of service profile. *
typedef struct RMW_PUBLIC_TYPE rmw_qos_profile_t
{
  enum rmw_qos_history_policy_t history;
  size_t depth;
  enum rmw_qos_reliability_policy_t reliability;
  enum rmw_qos_durability_policy_t durability;
  struct rmw_time_t deadline;
  struct rmw_time_t lifespan;
  enum rmw_qos_liveliness_policy_t liveliness;
  struct rmw_time_t liveliness_lease_duration;
  bool avoid_ros_namespace_conventions;
} rmw_qos_profile_t;
```


## Some useful QoS parameter Definitions

---
###Definitions
    - History Policy
        - The history policy determines how messages are saved until taken by the reader.
        - KEEP_ALL saves all messages until they are taken.
        - KEEP_LAST enforces a limit on the number of messages that are saved, specified by the "depth" parameter.

    - depth

    - Reliability Policy
        - The reliability policy can be reliable, meaning that the underlying transport layer will try to ensure that every message gets received in order, or
        - best effort, meaning that the transport makes no guarantees about the order or reliability of delivery.

    - Durability Policy

    - Liveliness Policy
---

###Posible values
Each of the policies within the rwm_qos_policy_t struct have the following possible options:

      - History Policy
          - RMW_QOS_POLICY_HISTORY_SYSTEM_DEFAULT
          - RMW_QOS_POLICY_HISTORY_KEEP_LAST		//# to keep defined in depth
          - RMW_QOS_POLICY_HISTORY_KEEP_ALL	// save all msgs until they are taken

      - depth = size of message queue if keep_last is chosen for history policy

      - Reliability Policy
          - RMW_QOS_POLICY_RELIABILITY_SYSTEM_DEFAULT
          - RMW_QOS_POLICY_RELIABILITY_RELIABLE
          - RMW_QOS_POLICY_RELIABILITY_BEST_EFFORT

      - Durability Policy
          - RMW_QOS_POLICY_DURABILITY_SYSTEM_DEFAULT
          - RMW_QOS_POLICY_DURABILITY_TRANSIENT_LOCAL
          - RMW_QOS_POLICY_DURABILITY_VOLATILE

      - Deadline (The period at which messages are expected to be sent/received)
          - uint64_t 	sec
          - uint64_t 	nsec

      - Lifespan (The age at which messages are considered expired and no longer valid)
          - uint64_t 	sec
          - uint64_t 	nsec

      - Liveliness Policy
          - RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT
          - RMW_QOS_POLICY_LIVELINESS_AUTOMATIC
          - RMW_QOS_POLICY_LIVELINESS_MANUAL_BY_NODE
          - RMW_QOS_POLICY_LIVELINESS_MANUAL_BY_TOPIC



# Predefined QoS Policies:

### Default RMW QoS Policy:
```cpp
static const rmw_qos_profile_t rmw_qos_profile_default =
{
  RMW_QOS_POLICY_HISTORY_KEEP_LAST,
  10,
  RMW_QOS_POLICY_RELIABILITY_RELIABLE,
  RMW_QOS_POLICY_DURABILITY_VOLATILE,
  RMW_QOS_DEADLINE_DEFAULT,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT,
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,
  false
};
```

### RMW Sensor policy

```cpp

static const rmw_qos_profile_t rmw_qos_profile_sensor_data =
{
  RMW_QOS_POLICY_HISTORY_KEEP_LAST,
  5,
  RMW_QOS_POLICY_RELIABILITY_BEST_EFFORT,
  RMW_QOS_POLICY_DURABILITY_VOLATILE,
  RMW_QOS_DEADLINE_DEFAULT,
  RMW_QOS_LIFESPAN_DEFAULT,
  RMW_QOS_POLICY_LIVELINESS_SYSTEM_DEFAULT,
  RMW_QOS_LIVELINESS_LEASE_DURATION_DEFAULT,
  false
};
```
> Notice that the Reliability Policy for Sensors (typically updated at a high frequency) is "Best Effort". You do not want to put this as reliable - as it can lead to network degradation

### More pre defined policies can be found [here](https://github.com/ros2/rmw/blob/master/rmw/include/rmw/qos_profiles.h)

---
# Code Examples

- Typically within the constructor of the class you've made for your node:

```cpp
...
      //
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



```
