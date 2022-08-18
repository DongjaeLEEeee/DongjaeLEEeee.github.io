---
layout: post
title:  "[Paper Review] Instantaneous Ego-Motion Estimation using Doppler Radar"
date:   2022-08-18 16:36:00 +
categories: Radar Ego-Motion-Estimation
---

D. Kellner, M. Barjenbruch, J. Klappstein, J. Dickmann and K. Dietmayer, "Instantaneous ego-motion estimation using multiple Doppler radars," 2014 IEEE International Conference on Robotics and Automation (ICRA), 2014, pp. 1592-1597, doi: 10.1109/ICRA.2014.6907064. [[Paper]](https://ieeexplore.ieee.org/document/6728341)

개요
----
Vehicle의 Ego-motion을 추정하기 위해서 흔히 wheel speed sensor, gyroscope, accelerometer 등을 사용한다. 그러나 wheel speed sensor는 vehicle의 kinematic imperfection으로 인한 systematic error, indeterminable interaction of the groud with the wheels (e.g. wheel slippage)로 인한 non-systematic error를 가진다. Accelerometer는 lower acceleration에서 낮은 signal-to-noise ratio를 가진다는 문제점이 있다. (Highly accurate gyroscope는 매우 비싸서 end customer application이 어렵다.)

이 논문에서는 stationary radar targets를 이용해서 ego-vehicle의 relative motion을 추정하는 방법을 제안한다. 논문에 따르면 clustering과 같은 별도의 preprocessing 작업 없이 single doppler radar의 measurement를 가지고 ego-vehicle의 velocity와 yaw rate를 추정할 수 있다. (This paper presents a robust and self-contained algorithm to instantly determine the velocity and yaw rate of the ego-vehicle. The algorithm is based on the received reflections of a single measurement cycle. It analyzes the distribution of their radial velocities over the azimuth angle. All targets are instantly labeled as stationary or non-stationary.)

Ego-Motion Estimation Algorithm
-------------------------------
For each measurement cycle three steps are performed.
1. Stationary Target Detection (RANSAC)
    + Largest Group of targets with the same movement (direction and velocity) is extracted.
    + It is assumed that there is no group of targets with exactly the same linear movement.
    + It is assumed that there is no larger number of targets than all stationary targets in common. (classifying all targets of the largest group found as stationary targets)

2. Velocity Profile Analysis
    + Movement of the radar sensor reconstructed by analyzing the returned radial velocity of all stationary targets with regard to their position in the azimuth angle.

3. Ego-Motion Estimation
    + Ego-motion of the vehicle is calculated from the sensor movement using the single-track model with the Ackermann condition.
    + Ackermann condition states that the extended rotational axes of all wheels of a vehicle intersect in one point.

<!-- ![Alt text](/assets/post1/figure1.png) -->
<img src="/assets/post1/figure1.png" width="30%" height="30%">


### Velocity Profile Analysis
Radar sensor가 움직일 경우, radar의 입장에서는 stationary targets가 opposite direction으로 움직이는 것처럼 보이는 것을 이용한다. All detected stationary targets의 radial velocity와 azimuth angle을 통해 sensor velocity와 heading direction을 reconstruct한다. 이때 key point는 stationary targets의 azimuth angle과 measured radial velocity가 sensor velocity를 amplitude로, heading direction을 phase offset으로 가지는 sinusoidal wave (velocity profile) 를 구성한다는 점이다. 총 N개의 stationary targets의 radial velocity와 azimuth position angle을 바탕으로 least-square approach를 사용하여 velocity profile을 계산한다.

<img src="/assets/post1/figure2.png" width="40%" height="40%">
<img src="/assets/post1/figure3.png" width="40%" height="40%">


### Ego-Motion Estimation
위에서 계산한 sensor velocity와 heading direction을 통해 vehicle의 ego-motion을 추정한다. 이때 ego-vehicle을 single-track model with the Ackermann condition로 모델링하는데, Ackermann condition을 가정함으로써 wheel drift는 없고 ego-vehicle의 velocity는 real axle에 orthogonal part만 존재하게 된다. (sensor position은 (l,b), mounting angle은 (Beta)) 

Single-track model with Ackermann condition과 vehicle의 ego-motion 식은 다음과 같다.

<img src="/assets/post1/figure4.png" width="40%" height="40%">
<img src="/assets/post1/figure5.png" width="40%" height="40%">


### Stationary Target Detection
Stationary targets를 통해 sensor velocity를 esitmate하고, estimated sensor velocity를 Single-track model with Ackermann condition을 가정하여 vehicle의 ego-motion으로 transform하는 것 까지가 위의 내용이다. 그렇다면 radar measurement 중에서 어떤 것이 stationary target인지는 어떻게 판별할까?
논문에서는 RANSAC을 통해 stationary targets를 identify한다. Velocity Profile Analysis를 하는 과정에서, RANSAC 알고리즘을 사용해서 모든 targets에 대한 best fitted velocity profile을 구한다. 이때 corridor threshold (+/- threshold) 를 사용해서 outlier (moving objects, clutter 등) 을 제거하고, inlier들을 stationary targets로 분류한다. (Error of the current fit is calculated by the sum of the radial velocity errors of all targets to the determined velocity profile of the current fit)

<img src="/assets/post1/figure6.png" width="40%" height="40%">
<img src="/assets/post1/figure7.png" width="30%" height="30%">

그러나, 이 방식은 group of objects which all have exactly the same linear movement and their number of received targets is larger than of the stationary objects 가 존재할 경우 fail한다. 이러한 문제점은 range of validity for the parameters of the velocity profile 을 사용하면 해결할 수 있다고 논문에서는 서술하고 있다. 



Conclusion
----------
이 논문에서 제안한 ego-motion estimation 방식은 아무 Doppler radar sensor에 적용할 수 있다는 장점이 있으며, radar sensor를 어디에 달던지 sensor의 position만 알고 있으면 vehicle의 velocity와 yaw rate를 추정할 수 있다. 또한 많은 수의 moving objects나 clutter가 있어도 stable하게 ego-motion을 추정할 수 있다. (그러나 이 경우 그보다 많은 수의 stationary targets가 measurement 안에 존재해야 한다.) 한계점으로는 single track model with the Ackermann condition에 의존한다는 점인데, 이는 multiple doppler radars를 사용하여 해결할 수 있다고 한다. [[pdf]](https://ieeexplore.ieee.org/abstract/document/6907064)