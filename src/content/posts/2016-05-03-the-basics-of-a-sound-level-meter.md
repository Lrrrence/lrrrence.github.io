---
title: 'The Basics Of A Sound Level Meter'
date: 2016-05-03
lastmod: 2022-08-28
tags: ['acoustic testing', 'Acoustics', 'audio measurement', 'environmental acoustics', 'SLM', 'sound', 'Sound Level Meter']
categories: ['Solent Acoustics']
---
The modern sound level meter is a powerful tool with many useful functions, but what are the most important things to know? This post aims to act as a simple to follow guide.

If you want to know how to use a sound level meter, you probably already know what you want to use it for, but here are some examples:

  * Measure Noise Levels 
    * Broadband or frequency dependant
    * Multiple parameters 
      * Min & Max
      * Average
      * Peak
    * Apply frequency weightings
  * Measure input from other transducers 
    * Accelerometers
    * Force Transducers
  * Measure reverberation time
  * Measure speech intelligibility
  * Function as an oscilloscope
  * Measure Total Harmonic Distortion (THD)
  * Record audio
  * Log data

### Calibration

For almost every scenario a sound level meter will need to be calibrated before use to ensure that it is reading the correct noise level, and that results will be comparable with others. In order to do this a calibrator must be used (which has also been calibrated previously) which produces a 1kHz tone at a level of 94dB. The sound level meter then measures the level of this sound using a connected microphone and adjusts its scale to match. If a different microphone is used the system will have to be recalibrated, as it is likely to have a different sensitivity, which also needs to be set prior to taking any measurements.

[![2016-05-03_13-15-03](/assets/img/wp-uploads/2016/05/2016-05-03_13-15-03.png)](<https://solentacoustics.wordpress.com/2016/05/03/the-basics-of-a-sound-level-meter/2016-05-03_13-15-03/>)

### Logging

Before taking any measurements you will need to setup a directory on your sound level meter for saving your results to. Different meters will have different options, some allowing for automatic logging after every measurement taken, while some are manual only. It is also possible on some meters to record the audio over each measurement period as a .wav file for subsequent analysis.

[![2016-05-03_13-17-00](/assets/img/wp-uploads/2016/05/2016-05-03_13-17-00.png)](<https://solentacoustics.wordpress.com/2016/05/03/the-basics-of-a-sound-level-meter/2016-05-03_13-17-00/>)

### Measurement Parameters

After calibration you are ready to take a measurement, but what parameters should you be interested in? Firstly you should determine what type of noise you have, is it constant or does it vary greatly? Are you interested in finding out just the level, or is the frequency content important to you? You can then pick the most applicable measurement parameters based on these answers.

There are 2 important parameters to determine for each measurement you take, the frequency weighting, and the time weighting.

#### Frequency Weighting

The frequency weighting relates to a correction that can be applied for measurements that involve humans. The human hearing system does not have a flat frequency response, instead it is most sensitive at mid frequencies where speech generally sits, and not so sensitivity at the low or high frequencies. A sound level meter does not hear like a human, and so applying a weighting allows us to account for this.

  * **Z - Weighting**
    * This refers to no weighting being applied. It should be used for any measurements that do no not involve human hearing. E.g. Testing the frequency response of a microphone.
  * **A - Weighting**
    * This should be used for most scenarios involving humans, but not when the level is very high (100dB+). E.g. Measuring the noise level from a festival at the nearest residential property.
  * **C - Weighting**
    * Again for use with levels affecting humans, but only for very loud noise. E.g. Measuring the noise levels at FOH at a music venue.

[![2016-05-03_13-18-27](/assets/img/wp-uploads/2016/05/2016-05-03_13-18-27.png)](<https://solentacoustics.wordpress.com/2016/05/03/the-basics-of-a-sound-level-meter/2016-05-03_13-18-27/>)

#### Time Weighting

The next parameter to select is the time weighting. This is where you will select how you want the noise to be measured. Some of the most common choices are shown below:

  * **Live Level**
    * If you are not interested in averaging over a time period, and just want to measure a level, you will select Live. You will then have a choice of F (Fast) or S (Slow) which refers to the integration time. If the noise is rapidly changing, using S will allow you to more easily gauge the level, but you may miss some peaks which would be clearly shown while using F.
  * **Min & Max**
    * These parameters are useful when you're only interested in measuring the minimum or maximum level of noise over a time period.
  * **EQ**
    * The total sound energy measured over a period of time. This is especially useful if you wish to know the average sound pressure level over a time period in which the noise varies. Many standard measurements are carried out using Leq.

Many sound level meters will allow you to display multiple different parameters on screen at one time, as well as saving many more to file.

[![2016-05-03_13-18-39](/assets/img/wp-uploads/2016/05/2016-05-03_13-18-39.png)](<https://solentacoustics.wordpress.com/2016/05/03/the-basics-of-a-sound-level-meter/2016-05-03_13-18-39/>)

The next step is to set the time period for your measurements. For continuous noise there is no need to measure for a long time period, but for random/impulsive noise it is best to measure for as long as possible (within reason), especially when using an Leq, so that you are recording an accurate representation of the noise level.

For some sound level meters the range will also need to be set, which will change depending on the average level of the noise that you are measuring.

You are now ready to take a broadband measurement, but what about if you wish to measure in the frequency domain?

### Real Time Analysis (RTA)

Many sound level meters are capable of analysing sound in the frequency domain in real time (RTA), which is useful for identifying more information about the noise than just a broadband level.

![2016-05-03_13-20-20](/assets/img/wp-uploads/2016/05/2016-05-03_13-20-20.png)

The same parameters as used in the broadband measurement can be used in RTA, although this time the frequency spectrum can be split into bands, and measurements taken at each of them. Commonly this is done over full octave bands (1/1) or 1/3 octave bands for greater frequency resolution. Some sound level meters will also allow for 1/12 octave band measurements.

**Check back again soon for a more in-depth look at other important measurement metrics!**
