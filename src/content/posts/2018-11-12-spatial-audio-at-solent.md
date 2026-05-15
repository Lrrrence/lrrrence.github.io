---
title: 'Spatial Audio At Solent'
date: 2018-11-12
lastmod: 2022-08-28
categories: ['Solent Acoustics']
---
Welcome to RM707, the newly configured spatial audio lab at Solent university! The loudspeaker installation here allows us to pan sounds around in full surround sound, using a system known as ambisonics. One of the major drawbacks to traditional surround sound formats (5.1, 7.1 etc.) is that they're channel based, meaning that the loudspeakers have to orientated in a particular way (to ITU spec), and when mixing audio around there are a very limited number of positions to work with. Ambisonics allows us to decode audio to any loudspeaker configuration, including loudspeakers positioned at different heights, as well as being able to pan sounds to positions in space rather than just to particular channels. 

### **The System**

To drive our system we have a Windows PC with PCI MADI card installed, connected to two Ferrofish 16 channel AD/DA Convertors, giving us a maximum channel count of 32. For loudspeakers we have 32 Genelec 8020c's, along with 4 Genelec 7050c subwoofers. Loudspeakers are positioned in 3 rings of 8, at 3 different heights, totaling 24. The 25th channel is used for the speaker directly above the listening position (“God channel”). The remaining channels up to 30 are currently unused, with 31 and 32 currently being used for subwoofers. Using the DAW [Reaper](<https://www.reaper.fm/>) we are able to configure the output channels as we wish, something that you're not able to do in the large majority of DAWs. To utilise ambisonics we need a decoder on the master channel that handles the playback based on the positional data of our loudspeakers, as well as an encoder on any audio tracks to allow it to be panned around.``

### **The Decoder**

For our channel count of 25 loudspeakers we can use a maximum ambisonic order of 3rd order. This accepts audio encoded in 3rd order (16 channels) and outputs it to our array. Using higher orders increases both positional accuracy and the size of the listening sweet spot. The decoder we’re using is [ambiX v0.2.8 built by Matthias Kronlachner](<http://www.matthiaskronlachner.com/?p=2015>) with a configuration file that has been generated using a Matlab script produced by the [Ambisonic Decoder Toolbox written by Aaron J. Heller](<https://bitbucket.org/ambidecodertoolbox/adt.git>). The script accepts loudspeaker positions as inputs (XYZ coordinates that are converted to radius, azimuth and elevation values) and outputs a configuration file for any ambisonics order or channel convention. ![WhatsApp Image 2018-10-25 at 11.06.01 PM](/assets/img/wp-uploads/2018/11/whatsapp-image-2018-10-25-at-11-06-01-pm.jpeg) The decoder uses an approach called “All Around Ambisonic Decoding” or AllRAD. This works by producing a decoder for a regular array using a large number of virtual speakers (around 5000) and then mapping them to the real loudspeaker array using Vector-Based Amplitude Panning (VBAP). In the image below the regular array created using virtual loudspeakers can be seen as circular black dots. The areas between each of them is joined up to create triangular tessellations, which are used to compute the VBAP gains for the real speaker array. The black squares represent the locations of the real loudspeakers. ![config](/assets/img/wp-uploads/2018/11/config-1.png) Plots produced by Aaron’s Matlab script give us an indication of how good the reproduction will be. There are two important factors in ambisonics that we can use to do this, RV and RE. 

  * RV – The velocity localization vector. This is the vector sum of the signals from the loudspeakers, it predicts low-frequency localisation (< ~800 Hz) based on Inter-aural Time Difference (ITD). If this value is equal to or close to 1, sounds will be precisely located.
  * RE \- The energy localization vector. This is the vector sum of the squares of the signals from the loudspeakers, it predicts mid-frequency localisation (> ~1200 Hz) based on Inter-aural Level Difference (ILD). If this value is equal to or close to 1, sounds will be precisely located.

For both RE and RV we can analyse the following factors: 

  * The magnitude of the vector, which indicates the quality or compactness of the phantom images in a particular direction.
  * Angular error – The angle between the real direction and RV or RE.
  * The direction difference (angle) between RV and RE, if these align then positional resolution will be good.
  * Pressure and Energy gains, which correspond to perceived loudness at low and mid/high frequencies.

The plots below show angular error for both RV and RE: The plot shows that for our setup angular error is low (good positional resolution) between -20 and +40 degrees, with a gap between +50 and +80 degrees. In future we may be able to increase the resolution in this area by adding in an additional 4 loudspeakers above the listening position. Below -20 degrees the angular error is high due to a lack of loudspeakers below the listening position. Due to the nature of ambisonics we are easily able to decode to any standardised format such as stereo or 5.1, as well as decoding to binaural for use with headphones. In future we may wish to increase the channel count (possibly introducing 4 speakers above the listening position rather than only 1) which can be achieved by generating a new decoder configuration file. 

### Encoding

We are able to use any 3rd order encoder with this system, most notably [O3A by Blue Ripple Sound](<http://www.blueripplesound.com/products/o3a-core>). Alternatives include the encoder bundled with the ambiX decoder, and [Bruce Wiggins’s WigWare plugins](<https://www.brucewiggins.co.uk/?page_id=78>). These encoders allow us to pan individual sounds to any location in space, based on azimuth and elevation values. ![panner](/assets/img/wp-uploads/2018/11/panner.png)

### Next...

The next stage of development for this facility is integration with VR, to have spatial audio in virtual reality has the potential to dramatically enhance immersion. Stay tuned for more on this in the future! ![HEADER-HTC-Vive-Pro-Full-Kit-SOURCE-HTC_topart](/assets/img/wp-uploads/2018/11/header-htc-vive-pro-full-kit-source-htc_topart.jpg)
