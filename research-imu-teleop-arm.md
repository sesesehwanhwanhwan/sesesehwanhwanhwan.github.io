# Wireless IMU Teleoperation of a Robotic Arm

*Solo semester project · Introduction to IoT and Engineering Design, Seoul National University · 2nd year*

<!-- HERO IMAGE: PPT p.3 right — the finished 3D-printed arm, or p.8 top-down shot of the full setup in use -->

## Overview

A wearable motion-capture controller that drives a 4-DOF robotic arm in real time over a
wireless link. Two MPU6050 IMUs are strapped to the back of the hand and the forearm; the
operator tilts their arm and the robot follows. A potentiometer mounted on a finger linkage
opens and closes the gripper.

The interesting problem was not getting a servo to move — it was **separating the motion of
the hand from the motion of the whole arm**, using nothing but two accelerometers and no
kinematic model.

Built end to end: 3D-printed arm, wearable sensor harness, firmware on both ends,
wireless protocol, and the signal processing in between.

## System Architecture

Two ESP32 nodes talking over **ESP-NOW**, a connectionless peer-to-peer protocol that needs
no router and adds minimal latency — important for teleoperation, where every millisecond of
lag is felt directly.

```
CONTROLLER (worn on arm)                    ROBOT ARM
┌──────────────────────────┐               ┌──────────────────────────┐
│ MPU6050 #1  (hand, 0x68) │               │ ESP32                    │
│ MPU6050 #2  (forearm,    │               │   ↓ I2C                  │
│              0x69)       │   ESP-NOW     │ PCA9685 16-ch PWM driver │
│ Potentiometer (gripper)  │  ─────────→   │   ↓                      │
│   ↓ I2C / ADC            │   9 bytes     │ 4× servo: base, arm1,    │
│ ESP32                    │   every 2 ms  │    arm2, wrist           │
│ 6 V supply               │               │ 1× servo: gripper        │
└──────────────────────────┘               │ 6 V supply               │
                                           └──────────────────────────┘
```

Both MPU6050s share one I2C bus, distinguished by tying the AD0 pin high on one of them
(0x68 / 0x69). The packet is a fixed 9 bytes: four `int16` accelerometer values plus one
`uint8` gripper angle.

<!-- IMAGE: PPT p.2 — the two wiring diagrams with the ESP-NOW arrow between them -->

## Hardware

**Robot arm** — 3D-printed, five servos: base rotation, two arm joints, wrist, and gripper.
The four arm servos run off a PCA9685 driver at 50 Hz; the gripper servo is driven directly
from an ESP32 GPIO.

**Controller** — worn, not held. The ESP32 and forearm IMU strap to the forearm with velcro;
a second unit sits on the back of the hand carrying the hand IMU and the gripper
potentiometer. Keeping both hands free was the point: the operator's arm *is* the input
device.

<!-- IMAGES: PPT p.3 (arm) and p.4 (controller: board → strapped to forearm → hand unit) -->

## Design Iteration: Flex Sensor → Potentiometer

The gripper was first driven by a flex sensor bent by the index finger. It proved too noisy
and too drift-prone to give a stable grip command.

It was replaced with a rotary potentiometer coupled to the finger through a small metal
linkage. A potentiometer measures absolute shaft angle directly, with no drift and a clean
monotonic ADC reading — a mechanically more complex but electrically far simpler answer.

<!-- IMAGE: PPT p.5 — flex sensor setup → potentiometer + linkage on hand -->

## The Coupling Problem

The first working version had a flaw that only appears once you wear it: **the robot
responded to the wrong motion.**

An accelerometer strapped to the hand measures the hand's orientation relative to gravity.
But that orientation changes both when the wrist bends *and* when the entire arm moves with
the wrist locked. One sensor cannot tell those two cases apart, so lifting the whole arm
produced the same command as bending the wrist.

The textbook answer is a kinematic model of the arm — link lengths, joint angles, forward
kinematics. For a one-semester project on two 8-bit-friendly microcontrollers, that was more
machinery than the problem deserved.

**The solution was to subtract one sensor from the other:**

```
New_Ax1 = Ax1 − Ax2
New_Ay1 = Ay1 − Ay2
```

This yields the hand's orientation *relative to the forearm*, which is what a wrist joint
physically is:

| Motion | Sensor 1 (hand) | Sensor 2 (forearm) | Difference |
|---|---|---|---|
| Whole arm moves, wrist locked | changes | changes equally | **≈ 0** — ignored |
| Wrist bends only | changes | unchanged | **changes** — commands the joint |

Whole-arm motion becomes common-mode and cancels; wrist motion survives. The joints that
needed to distinguish the two are driven from these difference terms, while the joints meant
to follow gross arm orientation read a single sensor directly. Two subtractions replaced a
kinematic chain.

<!-- IMAGE: PPT p.7 — the hand diagram with sensor 1 / sensor 2 axes -->

## Noise and Signal Processing

The second problem found in testing: small involuntary hand tremor made the arm visibly
shake and jump. The raw accelerometer signal carries the intended motion and the tremor and
electrical noise in the same channel, separated only by how fast each one changes.

An **exponential moving average** low-pass filter is applied at both ends of the link:

```
Transmitter:  fx[n] = 0.2·x[n] + 0.8·fx[n−1]
Receiver:     fx[m] = 0.3·x[m] + 0.7·fx[m−1]
```

Each stage is a first-order IIR filter costing one multiply-accumulate and one stored float —
cheap enough to run inside a 2 ms loop. In cascade:

```
                0.06                             α₁ = 0.2  (pole z = 0.8)
H(z) = ──────────────────────────                α₂ = 0.3  (pole z = 0.7)
        1 − 1.5 z⁻¹ + 0.56 z⁻²                   fs = 500 Hz
```

| Property | Value |
|---|---|
| Cutoff frequency | 14.0 Hz |
| Group delay | 12.7 ms |
| DC gain | 1 (no steady-state bias) |
| Poles | z = 0.8, 0.7 — both real → overdamped, no overshoot |
| Attenuation at 5 Hz (intentional motion) | 95% passed |
| Attenuation at 250 Hz (noise) | 2% passed |

Both poles are real and inside the unit circle, so the arm settles onto a new position
without ringing past it — the response is smooth rather than springy. The 14 Hz corner sits
above the bandwidth of deliberate arm motion (1–5 Hz) and below the tremor and sensor noise,
which is why the shake disappears while the intended motion does not.

![Filter characteristics](ema_filter_analysis.png)

![Raw vs. filtered signal on each axis](panel_all.png)

The filtered trace (blue) sits right on top of the raw signal (orange) through the slow, large
swings — the deliberate arm motion in the 1–5 Hz band is followed almost without lag. Where the
raw signal breaks into fast, jagged spikes, the filter rounds them off: the high-frequency tremor
and sensor noise are smoothed out while the underlying trajectory is left intact.

![Zoom on the first 1.6 s (ax axis)](panel_quiet.png)

Zooming in on just the first 1.6 s of the ax axis makes the effect clear at close range: the raw
signal (orange) is covered in a fine sawtooth jitter, while the filtered signal (blue) traces the
same path as one continuous line. The slow rises and dips are preserved; only the tremble riding
on top is taken out.

Output is further conditioned by a deadband near the neutral position and a slew-rate limit
of ±10 PWM ticks per update (≈ 2.2° of hand tilt), which caps how fast a servo can be
commanded to move without limiting normal operating speed.

## Control Mapping

The filtered accelerometer value is mapped linearly onto the servo pulse width:

```
PWM = map(fx, −15500, 15500, 100, 600)  =  fx/62 + 350
```

Under quasi-static conditions the accelerometer reads the gravity component, `a = g·sin θ`,
so the effective relationship between hand tilt and servo position is

```
PWM ≈ 264·sin θ + 350        ≈ 4.6 PWM ticks per degree of hand tilt
```

Note that this maps `sin θ`, not `θ` — the mapping is deliberately linear in the raw sensor
value rather than inverting the trigonometry, which keeps the receiver's work to a single
`map()` call. The cost is a gradually falling sensitivity at large tilt angles.

<!-- VIDEO: PPT p.6 and p.8 — arm following the hand; pick-and-place demo -->

## Limitations

- **Power.** Four AA cells sagged after 1–2 hours and the servos lost torque under load. A
  LiPo pack is the correct supply for five simultaneously loaded servos.
- **Potentiometer mounting.** The linkage worked but the electrical connection was fragile
  and needed to be glued down. A proper mount or a magnetic encoder would be the real fix.
- **Accelerometer only.** The MPU6050's gyroscope was never read. Using the accelerometer
  alone means orientation is inferred purely from gravity, which holds only while motion is
  slow; fast movement injects real inertial acceleration and corrupts the estimate. Fusing
  the gyroscope through a complementary or Kalman filter would remove that restriction.
- **No true orientation estimate.** Raw sensor values are mapped straight to servo positions
  rather than converted to angles, which is why the transfer is nonlinear in `sin θ`.

The last two are the reason this project pushed me toward IMU sensor modelling and sensor
fusion as a research interest.

## Possible Extensions

The same two-IMU relative-sensing setup generalises beyond arm teleoperation — gesture and
sign-language recognition being the most direct application, where the quantity of interest
is again the *relative* pose of hand and forearm rather than either in isolation.

## Presentation

<!-- LINK: original course presentation PDF -->
[Course presentation (PDF)](/files/imu-teleoperation-presentation.pdf)
