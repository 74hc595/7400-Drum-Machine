7400 Series Drum Machine
========================

This is a four-channel, sixteen-step percussion synthesizer/sequencer created entirely from discrete logic chips(mostly 74HC-series parts). That's right, no microcontroller or software of any sort. It was created for the Dangerous Prototypes 7400 Contest in a little over one month of spare-time work.

Photos: http://www.flickr.com/photos/74hc595/sets/72157627942539702/
Video: http://www.youtube.com/watch?v=QsSKh7Z2EVs

Sound generation
================
Sound is generated by four identical boards, one per voice. Each sound board has the following capabilities:

* Adjustable pitch and duration
* Four waveforms: square, sawtooth, triangle, noise
* Decay envelope
* Downward pitch bend

For the sake of the contest, the sounds are generated with digital circuits and a little analog glue. One half of a 556 dual timer is used in astable mode to generate an audio clock signal. Frequency is adjusted with a potentiometer, and a switch allows selection of two frequency ranges by connecting the threshold pin to either an 0.01 uF capacitor or an 0.001 uF capacitor.

The other half of the 556 is used in monostable mode and generates the attack pulse. A potentiometer controls the length of the pulse (and thus the duration of the sound.)

The audio clock feeds into a 74HC191 4-bit counter, and a binary weighted resistor DAC is used to convert the 4-bit value into an analog amplitude. This allows the creation of sawtooth and triangle waves.

A rotary switch selects the waveform. In square wave mode, only the most significant bit of the counter is used. In sawtooth wave mode, the 74HC191 is set to count upward, and the output of the DAC is a 4-bit sawtooth wave with positive slope. In triangle wave mode, the '191 alternates between counting upward and downward, by toggling a 74HC74 flip-flop connected to the DOWN/UP pin. Note that this causes the frequency of the triangle wave to be halved.

The other half of the 74HC74 dual flip-flop is used to sample a 1-bit noise source. The noise source is shared between all four sound boards and is simply an inverter with its output fed back to its input (a ring oscillator; more on this later.) The audio clock controls the sample rate, so the "pitch" of the noise can be controlled.

Another 74HC191 is used as an envelope generator. Since most percussive sounds have an instant attack, the envelope generator only provides a decay. This is where things start to get tricky. An LM324 quad op amp is used as a 4-bit digital volume control. But how is this possible!?

The output of the volume control is another 4-bit binary weighted DAC. But in this case, the input "bits" are not digital highs and lows, but the analog oscillator waveform replicated 4 times and buffered through op amps. Each "copy" of the signal is scaled down by a different factor, and their amplitudes are all added together at a summing junction.

Each op amp in the LM324 is set up as a differential (subtracting) amplifier. When an op amp's inverting input (one of the 4 control bits) is 0, the output voltage gets saturated at the negative rail (0 volts, since it's a single-supply op amp), effectively turning off that component.

Additionally, the 4-bit output of the envelope counter is passed through another binary weighted DAC. When pitch bend is on, this value is connected to the control pin of the audio half of the 556. Higher voltages on the control pin result in lower output frequencies, so the positive-going signal from the '191 causes the oscillator's pitch to decay.

The sound effect is activated when the TRIGGER input is pulled to ground. Each sound board also has a manual trigger input connected to a button on the front panel. Since the 74HC191 counters' reset inputs are level-sensitive, not edge-sensitive, an ac-coupling capacitor and pull-up resistors are used to generate a quick pulse when TRIGGER is pulled low. A diode provides protection from the voltage spike generated when TRIGGER goes high again, but in practice I've found things to be OK without it.

Phew, that was quite a lot! And we're just getting started!

Interface
=========

The front panel was designed in Inkscape, printed on a laser printer and glued to a piece of black foam core. (Who needs Ponoko? :p) At the top are the controls for the four sound generators. At the bottom are 16 switches and LEDs, one for each step, a rotary switch that selects the channel to edit, a master clear button, a start/stop button, and a potentiometer to control the tempo.

The 16 step keys are connected in a matrix configuration to a 74C922 keypad encoder. This chip is rare and *expensive* but it's awesome! It scans a 4x4 switch matrix, performs debouncing, and provides a 4-bit output. Sweet!

A 74HC154 is used to multiplex the LEDs. Only one LED needs to be lit at a time, so a single 33 ohm resistor is used for all the LEDs. During playback, the LED for the current step flickers. This is accomplished by using a 74HC85 4-bit comparator to compare the current step number to the multiplexer count. If they're equal, that LED's value is toggled using an XOR gate. (74HC86)

A 555 is used as a push-on/push-off circuit for the start/stop button. An RC circuit ensures it always starts off.

A 2-bit output is derived from the channel select rotary switch using some clever wiring. It's a 3-pole, 4-position switch, and two of the poles are used: one for bit 1 and one for bit 0. For each pole, the appropriate contacts are grounded; add pull-up resistors and you've got a 2-bit encoder!

