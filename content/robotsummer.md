---
title: "Robot Summer"
date: 2024-01-13T13:37:58-08:00
draft: false
tags: ['electronics', 'university', 'robotics']
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

### Chassis






![img](/gabespace/firstprototype.jpg)

