---
layout: page
title: Lab 5
---

## Lab 5:

### Prelab:

To control the PWM signals on my car, I wrote 2 different commands: 

1. STOP_MOTORS: stops the car completely 

2. SET_PWM: takes in 3 arguments (Motor, Forward/Backward, PWM value)

| Argument   | Associated Values |
|------------|-------------------|
| Motor      | 1: left, 2: right |
| Forward/Backward| 0: backwards, 1: forwards |



### Lab:

PWM = Kp (x-x_d)
difference is given in mm
(x-x_d) max at start, which is 2.4 to 4 m 

if BLE disconnects, STOP!


if the range is larger than 2.4, discard reading 


P/I/D discussion (Kp/Ki/Kd values chosen, why you chose a combination of controllers, etc.)
Range/Sampling time discussion
Graphs, code, videos, images, discussion of reaching task goal
Graph data should include Tof vs time and Motor input vs time (and whatever helps with debugging)
(5000) Wind-up implementation and discussion