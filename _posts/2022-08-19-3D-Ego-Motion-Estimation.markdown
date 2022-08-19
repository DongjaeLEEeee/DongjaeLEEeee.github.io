---
layout: post
title:  "[Paper Review] 3D Ego-motion Estimation using Low-cost mmWave Radars via Radar Velocity Factor for Pose-graph SLAM"
date:   2022-08-19 +
categories: Radar Ego-Motion-Estimation
---
<script type="text/javascript" src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS_HTML"></script>

Y. S. Park, Y. -S. Shin, J. Kim and A. Kim, "3D ego-Motion Estimation Using low-Cost mmWave Radars via Radar Velocity Factor for Pose-Graph SLAM," in IEEE Robotics and Automation Letters, vol. 6, no. 4, pp. 7691-7698, Oct. 2021, doi: 10.1109/LRA.2021.3099365. [[Paper]](https://ieeexplore.ieee.org/document/9495184)  
&nbsp;

개요
----
[지난번](https://dongjaeleeeee.github.io/radar/ego-motion-estimation/2022/08/18/Instantaneous-Ego-Motion-Estimation.html)에 살펴봤던 Instantaneous Ego-Motion Estimation using Doppler Radar 논문에서는 Doppler radar를 이용하여 ego-vehicle의 2D ego-motion을 추정한다. 박영상 박사님의 3D Ego-motion Estimation using Low-cost mmWave Radars via Radar Velocity Factor for Pose-graph SLAM 논문에서는 unique sensor system (two orthogonal mmWave radar sensors) 을 통해 3D velocity estimation을 하는 방법을 제안한다. 논문에 쓰여져 있는 본 논문의 contribution은 다음과 같다.

1. We devise a unique hardware configuration by combinint two low-cost Doppler radars and estimate 3D instantaneous velocity. This dual orthogonal configuration secures more returns from ground static objects. Also by tangential motion refinement, we substantially improved the performance over the single sensor case in the existence of dynamic objects.
2. The module was enclosed in a box as a realistic solution for a harsh environment. Although the casing reduced radar measurement quality, the proposed velocity-based solution presented outperformance over other methods.
3. We design a radar velocity factor for pose-graph SLAM and complete a 3D ego-motion in the integration with IMU (leveraging the rotation from IMU).  
&nbsp;

3D Velocity Estimation
-------------------------------
<img src="/assets/post2/figure1.png" width="60%" height="60%">  
&nbsp;

### Integrated Radar Module
Two orthogonal Doppler radars를 활용하여 3D linear velocity를 추정한다. 두 레이더 센서를 orthogonal하게 부착하여, 각각의 레이더에서 2D linear velocity를 얻고 이를 결합하여 3D linear velocity를 얻는다. 이때 두 레이더 센서는 플라스틱 박스 안에 들어가 있어, harsh한 환경에서도 사용가능하도록 세팅하였다. Plastic box로 casing을 하면 sparsity가 증가하지만, 여전히 ego-motion estimation은 성공적으로 가능하였다고 한다. (casing effect에 대해 설명하고 있는 figure를 보면 observed points가 wall에 제대로 찍히지 않는 것을 확인할 수 있는데, ego-motion estimation에는 range가 아닌 doppler effect를 이용하여 측정한 velocity만을 사용하므로 ego-motion estimation에는 문제가 없다.) 본 논문에서 두 레이더가 asynchronous한 문제는 별도의 external trigger circuits (Arduino) 를 사용하여 sensor data의 sync를 맞춰주어 해결하였다.  
&nbsp;

<img src="/assets/post2/figure2.png" width="50%" height="50%">
<img src="/assets/post2/figure3.png" width="33%" height="33%">  
&nbsp;

### 2D Ego-motion Estimation using Doppler Radar
Doppler radar를 이용한 2D Ego-motion estimation 방식은 Dominik Kellner의 논문에서의 방법과 동일하다. RANSAC을 사용해서 radar measurement 중에서 static object의 point들만을 추출하고, 이를 통해 2D linear velocity를 추정한다. 본 논문에서 사용한 2개의 mmWave radar 중 forward-looking radar를 통해 xy velocity를 얻고, upward-looking radar를 통해 yz velocity를 얻는다. (upward-looking radar는 하늘을 바라보게 달았다는 것이 아니라, 바닥을 바라보는 radar를 말하는 것 같다.)

<img src="/assets/post2/figure4.png" width="60%" height="60%">  
&nbsp;

### 3D Ego-motion by Two Orthogonal Radar Sensors
이제 각각의 radar에서 얻은 2D velocity를 concat해서 3D velocity로 확장한다. forward-looking radar를 통해 측정한 N개의 inlier points의 radial velocity와 azimuth angle, upward-looking radar를 통해 측정한 M개의 inlier points의 radial velocity와 azimuth angle를 아래의 행렬식으로 나타낼 수 있고, least square optimization을 통해 3D linear velocity를 계산할 수 있다. 그러나 이러한 naive한 integration 과정으로는 velocity estimation이 incorrect할 가능성이 크기 때문에, 후술할 tangential motion refinement 과정을 거쳐 velocity를 refine 해주어야 한다고 논문에서는 말하고 있다.

<img src="/assets/post2/figure5.png" width="60%" height="60%">  
&nbsp;

### Tangential Motion Refinement
Tangential motion refinement를 하기 위해서, 논문에서는 points with very subtle radial velocities에 주목한다. Radar는 radial 방향의 relative velocity만을 측정할 수 있으므로, subtle한 radial velocity를 갖는 point는 tangential velocity로 velocity가 주로 구성되어 있을 것이다. (ego-vehicle이 움직이는 상황에서 static object의 radial velocity가 subtle 한 상황은, 바닥을 바라보는 radar를 생각하면 된다. 바닥을 바라보는 radar의 경우 바닥은 정지해있고 ego-vehicle은 움직이므로 tangential 방향의 velocity만 주로 나타나게 된다.) points with very subtle radial velocities의 radial velocity direction들을 모두 모으고, dominant radial velocity direction을 계산한다. radial direction을 계산했으므로, 자연스럽게 tangential direction (two candidates in opposite direction) 도 결정된다. 이전 단계에서 estimate한 3D velocity를 dominant tangential velocity direction에 projection 시킴으로써 tangential motion을 refine 한다.

<img src="/assets/post2/figure6.png" width="60%" height="60%">  
&nbsp;

Radar Velocity Factor
-------------------------------
New radar linear velocity factor를 정의하고 GTSAM을 사용하여 full 3D ego-motion을 완성하였다. 새롭게 정의한 radar linear velocity factor에 IMU가 주는 rotation 정보를 합치면 기존의 IMU factor 대신에 pose-graph SLAM에서 사용할 수 있다는 것으로 이해하였다.

<img src="/assets/post2/figure7.png" width="60%" height="60%">  
&nbsp;