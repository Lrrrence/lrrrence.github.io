---
title: 'Guitar Soundboard Measurements For Repeatable Acoustic Performance Manufacturing'
date: 2018-11-20
lastmod: 2022-08-28
tags: ['guitars', 'reproduced sound', 'sine sweep']
categories: ['Solent Acoustics']
---
The following post gives a brief summary of a research paper submitted to [Reproduced Sound 2018](<http://reproducedsound.co.uk/>) primarily written by Ludovico Ausiello, with contributions from Lawrence Yule, Giacomo Squicciarini, and Chris Barlow.

A system for performing fast, accurate and objective assessment of the time-frequency response of guitar soundboards has been developed, using an application of the sine-sweep method commonly used to retrieve impulse responses of acoustic spaces or electro-acoustic devices.

Due to intrinsic characteristics of wood, namely its anisotropy and the relatively large variation of its key parameters such as density, stiffness and damping, guitars built to the same physical dimensions and using the same materials will have varying soundboard responses. Using the newly developed measurement system it may be possible to monitor the production of newly manufactured soundboards, both in terms of quality control, and as part of the manufacturing process.

Instruments could be graded based on whether they fit within certain tolerances, or a manufacturer could experiment with new designs while controlling acoustic performance. The system could be used throughout construction, on the soundboard only, with the addition of braces, the bridge, and finally with string tension, to see how each additional part changes the response of the guitar. This approach can also be applied to almost any acoustic instrument.

## Sine Sweep Method

The sine sweep method is a general purpose measurement methodology that can assess all types of linear and non-linear (although time-invariant) systems.

The Device Under Test (DUT) is stimulated with an exponential sine sweep, which is a signal of constant amplitude and exponentially rising frequency. A test signal has been created using the Aurora audio plugins developed by Farina, running on a Digital Audio Workstation (e.g. Audacity or Adobe Audition 3.0). When a sine sweep is created, its corresponding inverse filter is also automatically generated.

![](/assets/img/wp-uploads/2018/11/fdg.jpg)Experimental Measurement Setup

When convolving the original sine sweep with its corresponding inverse filter, a Dirac-like function (with its intrinsic bandwidth limitations) can be retrieved. When injecting a sine sweep into a DUT, the real system is expected to produce an output in forms of vibrations or acoustic waves. This output has been recorded and then convolved with the original inverse filter. The convolution product results in the DUT’s impulse response. Afterwards, FFT analysis can be applied to the computed impulse responses to analyse the response in the frequency domain. 

![](/assets/img/wp-uploads/2018/11/dfgdf.png)Electromechanical Exciter

An electromechanical exciter was used to inject energy into the DUT by attaching the voice coil to the guitar's bridge. A microphone (Earthworks MD30) was used to capture the response of the soundboard.

## Results

Measurements were carried out on 3 guitars for comparison, the instruments were produced in 2017; steel string, 00 size, 12 fret mahogany neck, and Sitka spruce tops:

  * Guitars A & B - Identical models (Blueridge BR-361), rosewood back and sides.
  * Guitar C - Blueridge BR-341, mahogany back and sides.

The FFT result for guitar's A and B are shown below:

![](/assets/img/wp-uploads/2018/11/ab.png)

While guitar A and B presented similarities in the 150-500 Hz region, a detailed view of the upper part of the response (below) was able to show substantial differences,which would be worth investigating from a manufacturing point of view.

![](/assets/img/wp-uploads/2018/11/hf-ab-1.png)

Guitar's A and C have also been compared:

![](/assets/img/wp-uploads/2018/11/ac-1.png)

The coupled main air resonance (seen at around 100 Hz) is clearly different between the two instruments, while it should be the same considering that this mode comes from the geometric shape and the body dimension.

## Conclusion

The method is capable of clearly identifying differences in the acoustic signature of the instruments, as well as being able to suggest classification criteria of their performance. Furthermore, the method was able to provide quantitative date to support subjective evaluation (and therefore marketing strategies) as well as relevant data that could be feed back to the R&D; process.

## Acknowledgements

The authors wish to thank **[Hobgoblin Music](<https://www.hobgoblin.com/>) **for understanding the strategical value of the presented idea, and supporting the research by offering all of the instruments used.
