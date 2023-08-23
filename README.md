## Before you continue

Dear Readers!

This is a fancier version of my summer project report; most of my content is just for fun. You will see some weird wording because I can’t write English and don’t know how to call the terms. The article is structured as an academic journal for my academic writing practice. Still, rather than using fancy words (which many academics do), I try my best to explain every single bit; anyone with a basic understanding of Electronics will have no trouble reading it. I hope even those people with no Engineering or Science backgrounds can also have a read and find something interesting through reading this article.
I am still writing this article. This is not a one or two-day job. There are many lab results and lab notes that are flying around everywhere. As a newbie in Analog Electronics, there are many phenomena that I cannot explain adequately, which requires me to consult a pro in that field to help me a little bit about that.
I hope you could have a good time reading this primitive, rough work with poor wording. But it would make me very happy if you could have fun or learn something after reading this article.

Best wishes,
Junzhe Chen
12th August 2023

Edit note:
- Resize figures
- Add more details onto conclusion and future work
- Add the picture of PCB when Martin comes back

## Abstract

## Introduction of PWM control

Greetings! In this blog, I will document the process of developing a fully discrete PWM Signal Generator with the ability to adjust reference voltage, output voltage, duty cycle and oscillation frequency. This blog aims to show how to make an analogue circuit project from scratch, how to test and troubleshoot the circuits and some potential future work that improves the circuit’s performance.
Before discussing the circuit, one question is why we want such a PWM generator?
PWM is used in many places that involve voltage control. For example, suppose you have a 12V DC motor and want to control how fast it spins. In that case, you cannot just apply a variable voltage onto it because the motor is 12V rated. Therefore, it requires 12V to make it spin. However, giving the motor 12V will make it spin too fast, and you probably don’t want that; that’s when PWM comes into use. Imagine if you turn the 12V voltage supply on and off periodically; the motor will spin for a while, then stop spinning, and so on. If I make the ‘turning on’ time longer relative to the ‘turning off’ time, the motor will spin faster and vice versa. If the switching frequency of this back-and-forth motion can be fast enough, the motor will spin smoothly. You can adjust the speed of the motor spinning by just controlling how much time you want the motor to stay on and how much time you want the motor to stay off, which controls the speed of the motor spin while driving the motor with the proper 12V power.

![application sketch](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/dbbea0f2-a9da-4b34-99c4-9943dd8cde35)

Now we understand the basic idea of how a motor spins, let’s see how we can implement it with some simple analogue tricks

## Closer look of PWM waveform

Example of SVPWM MATLAB code is provided in this [link](https://github.com/Junzhe-Chen/PWM-modulator-demo)

## Big picture view of the system

To make the PWM generator, the system can be broken down into the following three parts:

- A periodic signal that sets the frequency of the PWM signal
- A user controllable signal that is used to control the duty cycle of the PWM signal
- Some control unit that determines when to put the signal high and when to put  the signal low
- Output buffer that brings the high impedance input into low impedance output, in order to connect to the external circuit

The overall structure of the system is shown in the diagram below:

![System level diagram](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/a81c9ea1-c30a-4965-8c62-a3c4c7b23b04)

Now we can go through each block of that system

### Sawtooth wave reference

A good way to set up the reference periodic signal is by using Triangle Wave. It is easy to implement by charging and discharging a capacitor, and the frequency can be controlled and calculated by finding how much current is dumped into the capacitor. Also, a great thing about the sawtooth reference is the rising edge is proportional with time (if implemented correctly), which can provide a way to control the duty cycle linearly.

### User controllable signal

If by comparing the signal with sawtooth wave, the user controllable signal is relatively easy to make. Just use a potentiometer and connect the ends to the input power and ground, then you get the adjustable voltage at the middle end of the potentiometer

### Comparator

What comparator need to do in this circuit is by comparing two input voltage. 

- If the input voltage is lower than the reference voltage, give a HIGH output
- If the input voltage is higher than the reference voltage, give a LOW output

The SPICE results showed below demonstrates the working principle of a comparator. The yellow trace is the reference sawtooth input and the blue trace is the adjustable DC input signal. Notice that when the DC input is higher than the sawtooth reference, the output (red trace) is pulled HIGH. When the DC input is lower than the sawtooth reference, the output is then pulled LOW.

![PWM generation principle](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/e993372a-2e8b-4d92-9536-52d0398e5d2c)

### Output buffer

From what I have shown in the previous session, the circuit is quite awkward because it uses a capacitor as the voltage reference. To make sure the output is not capacitive, a buffer is much needed to transfer the high capacitive input impedance to a resistive low output impedance to drive the control circuit (for example, the gate of the control MOSFET). There are many ways to implement the buffer, a simple Source Follower (or Emitter Follower, for your BJT fans) should do the job fine, or a push-pull output pair if you want to be fancy or to generate PWM wave with dual voltage rail.

## Circuit Building Blocks Theory Implementation

### Sawtooth Wave Generator using Thyristor

There are many way and many circuit that can be used to generate triangle wave or sawtooth wave. One common traits of these circuits is they all have certain capacitor charging action. By studying the basic characteristics of the capacitor, we know that

$$
Q=CV
$$

Which 

- Q is the total charge accumulate on the capacitor
- C is the capacitance, which is the measure of how much charge this capacitor can store
- V is the voltage across the capacitor

Differentiate that equation, we obtain

$$
 \frac{dQ}{dt} = C\frac{dV}{dt}
$$

Electric current is defined as the charge flow over the time, which the horrible equation $dQ/dt$ simply means the current passing through the capacitor. By knowing this, the equation can be rewrite as

$$
I=C\frac{dV}{dt}
$$

which indicates that the rate of change in voltage $dV/dt$ across the capacitor is proportional to the current that passes through the capacitor.

Therefore, our idea of the sawtooth generator becomes clear. The principle is to charge the capacitor with a adjustable current source, and discharge the capacitor when the voltage across it reaches to certain threshold. 

![Sawtooth wave generation principle](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/2f89f043-b41a-43dc-bfdc-0127c2c98276)

Now we get the general idea done, the question is 

> How should we implement the transistor circuit to build those blocks?
> 

Building an adjustable current source is relatively easy, in this case a PNP transistor or a PMOS that dump the current into the capacitor should work just fine. The difficult part is to implement the control switch, which might not sound intuitive as it is

![Sawtooth wave generation circuit](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/fc793726-f649-4099-b80f-3f7c8b786eba)

In the circuit showed above, the left hand side is a classical current source that dumps current into the capacitor. The Zener diode $D_z$ sets up the gate bias voltage to turn the transistor on, current limiting resistor $R_{lim}$ is used to limit the current that passes through the Zener diode to reduce the power consumption of the circuit. Adjusting the value of $R_{I_{adj}}$ can adjust the current of the current source, so that it change how fast it charges the capacitor, thus adjust the frequency of the sawtooth wave.

The right hand circuit is relatively more confusing, the cross-connected transistor $Q2$ and $Q3$ are connected in a “Thyristor” configuration. It is basically a small positive feedback circuit and I will take further analysis of that part of the circuit below:

- When the voltage across the the emitter of PNP transistor $V_{in}$ is lower than the threshold voltage $V_{thr}$ plus the voltage at the collector of $Q3$, the transistor $Q2$ will be in saturation, therefore no current will flow through $Q2$
- When the voltage $V_{in}$ is higher than the threshold voltage $V_{thr}$ plus the voltage at the collector of $Q3$, the PNP transistor $Q2$ will be working at the forward active region, therefore current will flow from the collector of the $Q2$ and into the gate of $Q3$, and the voltage across the gate of $Q3$ will be one diode drop than the input voltage $V_{in}$, which will turn the transistor $Q3$ into forward active region, thus both $Q2$ and $Q3$ will be both turned on.
- Therefore, the capacitor will be grounded by transistors $Q2$ and $Q3$ and will be discharged.

The simulation results will be shown below:

![Sawtooth wave generation SPICE](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/1b65d438-4092-4d5d-ac37-3a55a2c21509)


*************************************? Wait, is the red waveform (Q2’s collector and Q3’s gate) indicates some sort of capacitor charging? Gotta ask some pros about this part.)*************************************

![Thyristor waveform](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/feb5c619-c6ff-4bb5-ab8d-620f25f1174b)

### Comparator design

As mentioned before, the comparator has to do few jobs for this design:

- If the input voltage is lower than the reference voltage, give a HIGH output
- If the input voltage is higher than the reference voltage, give a LOW output

There are many ways to implement this circuit. For example, using an Op Amp with open loop feedback. Because the gain of Op Amp is relatively large, in theory it should do the job. By using an uA741 from the lab, I obtained the following waveform from the scope

![Comparator with uA741](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/b0d23456-1960-4cc6-9e1d-245096a7d094)

That is NOT good AT ALL! The slew rate of the uA741 is quite slow (0.5V/us) which is not that great. Maybe understanding the working principle of the comparator and build it with discrete parts will achieve the better results. I will document the process later in the “Process” chapter

There are various ways to design a comparator, some are more complex than others. For a introductory circuit made by a noob like me, I am mainly looking at the open loop Long Tail Pair (LTP) **Differential Amplifier** with large Open Loop Gain ($A_{OL}$) and **Cross Coupled Pair** (XCP) with hysteresis control that provides positive feedback.  

![LTP basic](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/011e8f73-6c4f-4bc0-b246-99c149d8ebda)

Replacing the collector resistor with current mirror can improve the circuit’s performance. 

![LTP BJT](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/dcfc3e4b-f29e-4110-b1aa-d5411b3c9347)

MOSFET version of the circuit, which generally performs better than the BJT version due to the high input impedance and negligible gate current. However, the gain of the MOSFET is not as high as BJT which make the slew rate and current driving ability lower than its BJT counterparts. 

![LTP MOSFET](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/6deb0e0b-251a-4fad-a72f-27ef804fd365)

### Output buffer design

From the previous design, it is clear that the sawtooth reference waveform is generated by charging and discharging the capacitor, which makes one end of the input signal have capacitive impedance. This impedance will vary depending on the frequency that I set the sawtooth wave to, i.e. higher frequency, lower the impedance. Besides getting more DC-like waveform, this is also why I want the frequency of the sawtooth wave to be high. However, this does not tell the whole story;. At the same time, in the rising edge of the operation, the differential LTP will perform OK. The falling edge involves the discharge of the capacitor, which the LTP with limited current sinking ability cannot discharge the capacitor quickly, resulting in a limited slew rate. This can be found in Allen’s book “*CMOS analog circuit design” p.468.*

According to Allen:

> If a comparator has a large capacitive load, the chances are that it is slew rate limited. Under these conditions, we will show several methods capable of driving large values of CII. The first method is simply to add several push–pull inverters in cascade with the output of a two-stage, open-loop comparator
> 

![Allen's comparator](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/07882568-ae0e-4a2b-8ef7-bad1c8398dc1)

There are many ways to make the buffer, one way is to buffer up the output from the sawtooth wave. I tried but unluckily, the sawtooth wave does seems to be appeared at all. The solution that Allen provides is to use two inverters as the output push-pull buffer shown below:

Figure from LTspice that shows the buffered effect

![Buffered waveform](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/0a64e5e2-65de-482e-8b0a-af45ade0b1b2)

Despite this circuit already have good buffering system, there is still one place that can further improve the circuit’s performance. Transistor M6 and M7 can be mirrored to the left hand side which the output would be taken as the voltage difference between the Drain of M1 and M2, double the voltage difference compared to the proposed circuit, as shown in the schematics below:

![My comparator](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/7248a9fa-23f5-4153-88b2-b51a588e34f3)

M14 and M15 are act as the complement circuit that amplifies the voltage difference between the drain of M4 and M5, which double the difference amplifier gain and improve the current driving ability.

## My process of the development

### Final Circuit Design Overview

Circuit schematics

![My circuit](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/b85c47aa-af63-4e80-90ae-759ec39633a9)

Output waveform from SPICE

![My circuit SPICE output](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/99cc9921-2cf9-498a-8455-c33cde343424)

### Results

Output waveform from the Oscilloscope

<img width="421" alt="My circuit real output" src="https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/499c9aaf-9bdd-427c-a324-ed484936237c">

Screenshot from the oscilloscope

![Oscilloscope screenshot](https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/204ccb69-5b3e-49a8-b458-e852abb49f38)

Slew rate measurement, **155V/us**

<img width="419" alt="Slew rate" src="https://github.com/Junzhe-Chen/PWM-Modulator-Circuit/assets/141964509/750d6102-e242-4241-b8e7-71f9529ffa7b">

## Conclusion and Future Works

### Conclusion

In this article, an PWM wave generator circuit with the following functions is being proposed:

- Sawtooth wave generator with adjustable frequency and amplitude
- Full range adjustable duty cycle from 0 to 1 by the use of potentiometer
- High slew rate in the switching edge of the PWM signal at 155V/us

However, it still have limitations listed below:

- Switching oscillation at the edge
- Phase shift at low duty cycle
- Weird artifacts at high frequency, upon to my test it works OK under 10kHz

### Future works

- Make it work at higher frequency
- Fix phase shift at low duty cycle
- mitigate oscillation
- explore the application of the PWM modulator and some of the building blocks. This will be used as part of my future Full Discrete DC-DC Buck Converter project
