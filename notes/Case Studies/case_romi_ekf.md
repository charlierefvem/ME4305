---
title: Romi Localization with an Extended Kalman Filter
type: reference
tags:
  - batteries
  - ADC
source:
  course: ME405
  term: 2262
status: dirty
---


The nonlinear model before augmentation is
$$
\begin{aligned}
\frac{\mathrm{d}}{\mathrm{d}t}
\begin{bmatrix}
\Omega_L \\
\Omega_R \\
s \\
\psi \\
X \\
Y
\end{bmatrix}
&=
\begin{bmatrix}
\frac{K_m}{\tau}\,u_L-\frac{1}{\tau}\Omega_L \\
\frac{K_m}{\tau}\,u_R-\frac{1}{\tau}\Omega_R \\
\frac{r}{2}\left( \Omega_R + \Omega_L \right) \\
\frac{r}{w}\left( \Omega_R - \Omega_L \right) \\
\frac{r}{2}\,\cos\psi\,\left( \Omega_R + \Omega_L \right) \\
\frac{r}{2}\,\sin\psi\,\left( \Omega_R + \Omega_L \right)
\end{bmatrix}.
\end{aligned}
$$

## Differential Drive Disturbance Model

The $\Omega_L$ and $\Omega_R$ states are left and right wheel velocities. The $s$ and $\psi$ states represent the total signed displacement (essentially arc-length) and the yaw-angle (not absolute heading) respectively. The $X$ and $Y$ states are the absolute location of the center of the robot.

The inputs $u_L$ and $u_R$ are the voltage inputs to the left and right wheel motors.

The outputs $s_{enc,L}$ and $s_{enc,R}$ are the displacements at the center of the left and right wheels, respectively, as measured by wheel encoders. The $\psi_{IMU}$ and $\dot\psi_{IMU}$ outputs are the yaw angle and yaw rate as measured by an IMU. The outputs $X_{GPS}$ and $Y_{GPS}$ are measurements of the robot's absolute location from a GPS module. The output $z$ is a pseudo-measurement defined by $z = \psi - \frac{1}{w} \left(s_{enc,R} - s_{enc,L}\right)$.

The $d_1$ and $d_2$ disturbance states handle disturbance torques for either motor, primarily due to inaccurate $K_m$. The $d_3$ and $d_4$ disturbance states account for wheel slip at the left and right wheels. The $b_1$ bias state represents constant bias in the IMU yaw angle.

The augmented system model including output equations is
$$
\begin{aligned}
\frac{\mathrm{d}}{\mathrm{d}t}
\begin{bmatrix}
\Omega_L \\
\Omega_R \\
s \\
\psi \\
X \\
Y \\
d_1 \\
d_2 \\
d_3 \\
d_4 \\
b_1
\end{bmatrix}
&=
\begin{bmatrix}
\frac{K_m}{\tau}\,u_L-\frac{1}{\tau}\Omega_L+d_1 \\
\frac{K_m}{\tau}\,u_R-\frac{1}{\tau}\Omega_R+d_2 \\
\frac{r}{2}\left( \Omega_R + \Omega_L \right)+d_3 \\
\frac{r}{w}\left( \Omega_R - \Omega_L \right)+d_4 \\
\frac{r}{2}\,\cos\psi\,\left( \Omega_R + \Omega_L \right) \\
\frac{r}{2}\,\sin\psi\,\left( \Omega_R + \Omega_L \right) \\
0 \\
0 \\
0 \\
0 \\
0 
\end{bmatrix}, \\
%
\begin{bmatrix}
s_{enc,L}\\
s_{enc,R}\\
\psi_{IMU} \\
\dot\psi_{IMU} \\
X_{GPS} \\
Y_{GPS} \\
z
\end{bmatrix}
&=
\begin{bmatrix}
s-\frac{w}{2}\,\psi \\
s+\frac{w}{2}\,\psi \\
\psi + b_1 \\
\frac{r}{w}\left( \Omega_R - \Omega_L \right) \\
X \\
Y \\ 
b_1
\end{bmatrix}.
\end{aligned}
$$
