---
title: "Robot Summer"
date: 2024-01-13T13:37:58-08:00
draft: false
tags: ['electronics', 'university', 'robotics']
math: true
---
# What exactly is robot summer?
Robot summer is a nickname for the summer course ENPH253 --- Introduction to Instrument Design. The course is exclusive to Engineering Physics students and, as the nickname would imply, is taken over (both) summer terms. Over a period of 6 weeks, students divide themselves amongst 16 groups of 4 to build a fully autonomous robot capable of completing some tasks in a competition. It was a very fun, intense and gratifying experience.

## The competition
For my year, the competition involved following black tape across the following multi-terrain track: 

{{< img src="/gabespace/track.png" >}}

The sections in blue are ramps, Rainbow Road is pretty aggressively arched.
Each lap across the track was worth 3 points, and there were blocks scattered around that we could pick up, each worth 1 point. This was the first time this competition allowed robots to run simultaneously on the same track, which added some interesting and challenging considerations. If the robot somehow can't keep going, you may restart it, forfeiting *all* your lap points, or call it day and keep the current tally.
***
# Our design
We came into this competition with the goal of winning, so we really tried to figure out an "optimal" strategy. We realized we should focus on the basics, *speed* and *consistency* (i.e. doing laps without restarting). This immediately removed a few strategies that may have seemed *really* good for saving time:

1. We would *not* take the IR shortcut: we realized it would be pretty hard to go fast across the rocks and track the IR reliably;
2. We wouldn not take the zipline: same reasons.

These both turned out to be really good insights. More details on the design:

### Chassis and mechanical design
I'll go into more detail on the chassis development later hopefully. My contributions here came
quite late into the course as the robot systems got more integrated.

### Electrical design
This was my main role during the first half of the course. I learned a lot about interfacing different
circuits here, so I'd like to briefly document my journey through this process. Below is the initial high-level
overview I proposed for the robot:

{{< img src="/gabespace/highlevelcircuits.png" caption="Disregard the IR, we immediately abandoned that idea.">}}

I chose to highlight shared grounds in blue here, and isolated/protected grounds in red. I'll go over each part of the system
in a second, but the big takeaways are: 

1. We chose to use one 8.4V and one 14.8V battery, this turned out to be a good choice as we ended up pretty tight for space and weight at
the end.
2. Big noise sources are isolated; this is because the microcontroller we're using is quite sensitive to noise.

The microcontroller of choice for this course is the STM32F103C8T6/8 (commonly referred to as the *Blue Pill* board). With our mechanical design, 
I determined the following pinout: 

{{< img src="/gabespace/bppinout.png" >}}

The required PWMs are mostly a result of our mechanical choices, we needed a minimum of two for the rear motors, another two for the motors in our
pickup mechanism (didn't change moving from servos to DC motors) and another one for the steering servo. I managed to save a some pins for the 
sonar with a clever little circuit I'll share later, but let's begin with the essentials:

#### The H-Bridge
The H-Bridge is a fundamental circuit used for switching the polarity of voltage across a load, commonly used to control the *direction* of
inductor driven DC motors. The design we used was:

{{< img src="/gabespace/hbridgeschem.png" >}}

Note the opto-isolators on the Blue Pill outputs (labelled `BP_OutX` here). The component that really stands out here compared to what happens if you toss "H-Bridge circuit" into google is the LTC1161 IC; this is called a "gate driver" and is really just a combination of fast switching capacitor charge pumps that output a high voltage (as in, beyond the rails). This allows us to use exclusively NPN transistors to drive both sides of the H-Bridge (literature usually uses PNP transistors at the top, driven by a low voltage signal). But one wonders, why limit yourself to only NPN MOSFETs? The main reason is they're simply easier to manufacture, and thus cheaper and more prominent in industry. For this use case, they're also decently faster, which is important when you're switching on and off rapidly (as a PWM does).

Implementing this circuit is *usually* pretty tricky. It's quite involved, noisy and prone to issues, fortunately we didn't have many issues with our initial proto-board implementation, other than it being pretty ugly and massive. The latter being a real issue, which is why I eventually designed and ordered some compact PCBs of this circuit.

The rest of this article isn't finished yet so here's a placeholder stylish lap: 

{{< img src="/gabespace/improvements.gif" >}}

