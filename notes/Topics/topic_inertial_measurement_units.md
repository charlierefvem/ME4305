---
title: Inertial Measurement Units (IMUs)
type: topic
tags:
  - imu
  - accelerometer
  - gyroscope
  - magnetometer
  - euler-angles
  - quaternions
  - rotations
source:
  course: ME 405
  term: 2262
  lecture: 16
status: draft
---

# Motivation

An IMU is useful because it helps a robot estimate its orientation. This is especially important when a robot needs to understand how it is rotated relative to some inertial or world reference frame.

However, IMUs can be misleading if we treat them as position sensors. They do **not** directly measure position, and trying to integrate accelerometer data twice to estimate position usually goes poorly. The more practical goal is to use the IMU to estimate orientation from a combination of accelerometer, gyroscope, and sometimes magnetometer data.

This lecture introduces the sensors inside an IMU and then reviews two common ways of representing 3D orientation.

# Inertial Measurement Units

IMU stands for **Inertial Measurement Unit**. An IMU measures orientation with respect to an inertial reference frame. Inertial reference frames are those that do not accelerate

IMUs are composed of multiple multi-DOF sensors. Common sensors include:

| Sensor | Typical data | Purpose |
|---|---:|---|
| Accelerometer | $(a_x, a_y, a_z)$ | Measures linear acceleration; often used to estimate the direction of gravity |
| Gyroscope | $(\omega_x, \omega_y, \omega_z)$ | Measures angular velocity, or the rate of 3D rotation |
| Magnetometer | $(B_x, B_y, B_z)$ | Measures magnetic field strength; often used as a compass |
| Barometer | $p_{\text{atm}}$ | Measures atmospheric pressure; often used as an altimeter |

Common shorthand labels include:
* **6 DOF IMU**: accelerometer and gyroscope
* **9 DOF IMU**: accelerometer, gyroscope, and magnetometer
* **10 DOF IMU**: accelerometer, gyroscope, magnetometer, and barometer

IMU orientation data can appear in several forms:
* Raw sensor data, usually 6 to 10 values depending on the sensor package
* Euler angles, such as yaw, pitch, and roll
* Quaternions, which represent one rotation axis and one rotation angle

## Accelerometers

An accelerometer measures linear acceleration:

$$
(a_x, a_y, a_z)
$$

In an IMU, accelerometers are often used to measure the direction of "down" by measuring the gravity vector. When the sensor is not accelerating significantly due to motion, the measured acceleration mostly reflects gravity, which gives information about orientation.

Digital accelerometers commonly use MEMS technology, short for **micro-electro-mechanical systems**. A simplified MEMS accelerometer can be understood as a small moving mass attached by a spring. As the mass deflects, the spacing between fixed and moving plates changes. The sensor measures this change in capacitance.

![A diagram showing the internals of an accelerometer including a moving mass supported by a spring with moving plates and fixed plates forming variable capacitors; acceleration causes the moving mass and plates to deflect relative to the fixed plates.](images/imu/accelerometer.svg)

Reference video from the slide: https://www.youtube.com/watch?v=9X4frIQo7x0

## Gyroscopes

A gyroscope measures angular velocity:

$$
(\omega_x, \omega_y, \omega_z)
$$

Gyro data is useful for tracking changes in orientation, but drift can cause large errors because angular velocity must be integrated over time to estimate angle. A small bias in angular velocity becomes a steadily growing angle error after integration. For this reason, gyroscope output must be calibrated.

Almost always, other sensors are used with a gyroscope for error checking or correction. In an IMU, the accelerometer and magnetometer can help correct the drift that would accumulate if the gyroscope were used by itself.

Similar to an accelerometer, a MEMS gyroscope actually measures an acceleration effect. In this case, it measures Coriolis acceleration, which is approximately
$$
\vec{a}_c \approx \vec{\Omega} \times \vec{v}
$$
where $\vec{\Omega}$ is the rotation rate and $\vec{v}$ is the velocity of the vibrating internal element.

## Magnetometers

A magnetometer measures magnetic field strength:
$$
(B_x, B_y, B_z)
$$

Magnetometer data can be used like a compass to find north. This gives an absolute orientation reference, which can be valuable because it does not require integrating angular velocity over time.

However, magnetometers can work poorly in dynamic environments, especially if there are nearby magnetic materials, motors, high currents, or changing electromagnetic fields.

Many magnetometers rely on the Hall Effect. In the Hall effect, current running through a conductor causes charge separation if a magnetic field is present.

![A diagram showing a conductor connected to a battery with conventional current, a magnet near the conductor, and charge separation across the conductor labeled negative on one side and positive on the other.](images/imu/hall_effect.svg)

## Orientation Representations

Recall that IMUs commonly produce 6 to 9 pieces of motion and field data:
* Accelerometer: $a_x, a_y, a_z$
* Gyroscope: $\omega_x, \omega_y, \omega_z$
* Magnetometer: $B_x, B_y, B_z$

These data can be fused and converted into an orientation estimate. The orientation can then be represented using either Euler angles or quaternions.

### Euler Angles

Euler angles represent an orientation as three sequenced rotations. There are many possible conventions, including `ZYX`, `ZYZ`, `XYZ`, and others. In total, there are 24 possible ordered conventions.

A very common convention is yaw, pitch, and roll, often represented as a `ZYX` sequence:

1. Yaw / heading: angle $\psi$ about the global $z$ axis
2. Pitch: angle $\phi$ about the new intermediate $y$ axis
3. Roll: angle $\beta$ about the new local/body $x$ axis

In the notation used here:
* $\psi$ is yaw / heading.
* $\phi$ is pitch.
* $\beta$ is roll.

Read more about [[reference_euler_angles|Euler Angles]].
### Quaternions

Quaternions represent an orientation using an axis and angle of rotation about that axis encoded by four numbers split into one "real" part and 3 "imaginary" parts, $i$, $j$, and $k$. Quaternions act like a higher dimensional version of complex numbers. Unlike Euler angles, which are applied in a three step sequence, quaternions apply as a single operation.

Read more about [[reference_quaternions|Quaternions]].
# The BNO055 IMU

The \[\[BNO055\]\] from Bosch is a 9 DOF IMU. It combines an accelerometer, gyroscope, and magnetometer in one sensor package.

The breakout board used in lab exposes power, ground, and I2C connections. The IMU chip itself is the small sensor IC on the board.

![A photograph of the BNO055 breakout board. Match the dot on the sensor to the dot in the coordinate diagram to find the axes.](bno055_breakout_board.png)

When using the BNO055, pay attention to the sensor coordinate axes. The axis definition depends on the physical orientation of the chip. The dot on the sensor package can be matched to the dot in the reference diagram to determine the sensor axes.

![Annotated BNO055  coordinate definition. Match the dot on the sensor to the dot in the diagram to find the axes.](bno055_axes.png)

Image sources from the slide:
* Adafruit: https://learn.adafruit.com/assets/24585
* Bosch BNO055 Quickstart Guide: https://www.bosch-sensortec.com/media/boschsensortec/downloads/application_notes_1/bst-bno055-an007.pdf

# Summary

This lecture introduced IMUs as sensor packages used to estimate orientation. A typical IMU combines accelerometer, gyroscope, and magnetometer data, and some packages also include a barometer. The accelerometer gives information about the gravity direction, the gyroscope measures angular velocity, and the magnetometer provides a compass-like reference to magnetic north.

IMUs are orientation sensors, not position sensors. The accelerometer is useful for determining the gravity direction, but integrating accelerometer data to get position is usually not a reliable strategy in this context.

A gyroscope provides excellent short-term rotation-rate information, but integration makes bias and drift accumulate over time. This is why IMU orientation estimates often combine gyroscope data with accelerometer and magnetometer data.

# See Also:
* [[topic_i2c_communication|Introduction to I2C]]
* [[reference_euler_angles|Euler Angles]]
* [[reference_quaternions|Quaternions]]