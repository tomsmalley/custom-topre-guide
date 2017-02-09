
# Guide for designing a custom Topre keyboard

## Circuitry

### Overview

The method I use to sense key depression is rather simple. In tests that I
have done it works well provided some calibration is performed in the firmware
to normalise the readings.

The matrix crossing points of a Topre keyboard are essentially variable
capacitors which connect a "strobe" line to a "read" line. The strobe and the
read lines form the electrodes directly on the PCB, and the conical spring
under the dome couples them together, creating a variable capacitor with the
range 0 ~ 6 pF (roughly). The strobe lines are just digital signals from a
digital out pin of the microcontroller. The read lines are dealt with in the
following schematic:

![Schematic](schematic.png "Basic schematic")

Each read line is pulled to ground with an individual 22k resistor, and fed
into an analog multiplexer. Any unused inputs of the multiplexer should be
grounded to prevent additional sources of noise: this goes for any unused op
amp pins too. After selecting a read line on the multiplexer, the
microcontroller strobes a column and a small voltage pulse can be seen on the
selected read line, larger pulses correspond to greater key depression.

The selected read line is connected to the capacitor C1, which causes the read
line to behave like a simple RC decay circuit. The value can be chosen given
the following formula:

```
                                       Capacitance of key we are sensing
Peak output voltage = Input voltage * -----------------------------------
                                             Total row capacitance
```

Here the input voltage is `Vdd`. The total row capacitance (to ground) consists
of C1 plus the capacitance of all the keys in the row. As such it is clear that
choosing a large value for C1 (compared to key capacitance) is important so
that our reading is not significantly altered due to other keys on the row
being depressed. We can't just make C1 enormous though, because it drops the
peak output voltage which ultimately contributes to a higher noise level. I
found 1 nF to be a good value.

Ignoring the "drain pin" for now, the read line passes through a current
limiting resistor into a non inverting amplifier. The purpose of this is to
provide a clean signal boost back into the range of 0 - 3.3V. The gain is given
by `1 + R2 / R4` which in this case is around 200. It also serves to protect
the microcontroller from negative voltages which can happen when the strobe
line returns to ground. I found it important to use a very fast amplifier here,
opting for the OPA350A. Cheaper options proved to be too slow, turning the
voltage spike into more of a voltage mound. The output of the amplifier should
connect to an ADC pin of the microcontroller.

### Drain pin

With the selected read line forming an RC circuit we can see that the time for
it to relax to ground is simply governed by `5 * RC time constant`. The time
constant is just `R * C`, which in our case gives a total relax time of
`5 * 22k Ohm * 1 nF ~ 100 us`. Bearing in mind that we must wait for the matrix
to relax to ground before reading the next key, this translates to taking 100
us per key of the keyboard - giving us a polling rate less than 1000 Hz for
keyboards with more than 10 keys. In order to fix this, we just connect the
read line to the pin of the microcontroller through a current limiting 1k
resistor R1. This pin should be floating during the strobe and read process,
but after we have captured the reading in the ADC (takes around 5 us)
it can be grounded, reducing the resistance R according to the parallel
resistor formula:

```
  1     1       1
 --- = ---- + -----
  R     R1     22k
```

which gives `R ~ 950 Ohms` for our chosen values. Recalculating the relax time
now gives `5 * 950 Ohms * 1 nF ~ 5 us` - much faster.


## Hardware (case/plate)


## Firmware


