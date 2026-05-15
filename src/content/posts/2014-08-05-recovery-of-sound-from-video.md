---
title: 'Recovery of sound from video'
date: 2014-08-05
lastmod: 2022-08-28
tags: ['camera', 'Frame Rate', 'frame rates', 'Massachusetts Institute of Technology', 'MIT', 'sound', 'Vibration', 'video']
categories: ['Solent Acoustics']
---
Research carried out by the Massachusetts Institute of Technology, along with Microsoft and Adobe, has been used to extract audio data from video by analysing the tiny, imperceptible vibrations that occur in objects when they are subject to a sound.

The system works best when used with high frame rate cameras, as the sampling rate of the video needs to be higher than the audio frequencies of interest. The researchers have also been able to show, however, that audio can still be recovered from standard frame rates (60fps), albeit at a lower audio quality.

[![Setup](http://solentacoustics.files.wordpress.com/2014/08/setup-e1407249425187.png?w=474)](<https://solentacoustics.files.wordpress.com/2014/08/setup.png>) Image: System setup (MIT) 

In order to carry out this process the video must be analysed in great detail. First, an area of the image that contains a clear colour boundary is identified. By looking at a boundary (such as the edge of an object) the minute changes in colour between pixels can be measured, which can in turn be related to the frequency of vibration that is travelling through the object. Unfortunately the vibrations are smaller than the width of a single pixel (around 1/10th of a micrometer), and so movement must be inferred by the tiny change in colour value between adjacent pixels over time.

By using a number of different image filters the small variations can become more apparent, while "fuzzy" edges between boundaries can be sharpened up, making the sound easier to recover.

The system works differently depending on the type of object being filmed, with the quality of the extracted audio being very much dependant on the amount of vibration passing through the medium, with many high density, absorptive materials (such as brick) limiting the propagation of high frequency sounds. See the image below for material examples, and listen to them [**here**](<http://people.csail.mit.edu/mrub/VisualMic/Supplemental/Chirp/index.htm>).

[![image3015](/assets/img/wp-uploads/2014/08/image3015.png)](<https://lawrenceyule.com/wp-content/uploads/2014/08/image3015.png>) Image: Comparison of audio recovered from different objects (MIT) 

When attempting to recover audio from video that has not been recorded at a high frame rate the researchers have made use of the rolling shutter found in most consumer grade cameras (and phones) which processes each line of pixels sequentially at slightly different time intervals. Normally this would create unwanted artefacts in the video when recording moving subjects, but by analysing each line individually in each frame the sampling rate is effectively increased, and frequencies above the frame rate of the camera can be recovered. The quality of audio is not as good as with a high frame rate camera, but the developers say that it could still be used to identify male or female voices, or the number of people present in a room, which could be potentially useful.

Alexei Efros, an associate professor of electrical engineering and computer science at the University of California at Berkeley had this to say about the research: “I’m sure there will be applications that nobody will expect. I think the hallmark of good science is when you do something just because it’s cool and then somebody turns around and uses it for something you never imagined. It’s really nice to have this type of creative stuff.” (_News Office, MIT_).

Watch a video of the process, courtesy of Abe Davis **[here](<https://www.youtube.com/watch?v=FKXOucXB4a8>).** Read the full paper **[here](<http://people.csail.mit.edu/mrub/papers/VisualMic_SIGGRAPH2014.pdf>)**.
