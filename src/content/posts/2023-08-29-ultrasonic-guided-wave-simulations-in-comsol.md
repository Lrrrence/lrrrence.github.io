---
title: 'Ultrasonic guided wave simulations in COMSOL'
date: 2023-08-29
lastmod: 2023-08-29
tags: ['Acoustics', 'COMSOL', 'Guided waves', 'Simulation']
categories: ['Acoustics']
---
There are a number of methods of simulating ultrasonic guided wave propagation in COMSOL (v6.1) which I will discuss in this article. Principly there are two physics interfaces available, Solid Mechanics and Elastic Waves, Time Explicit. 

The Elastic Waves, Time Explicit physics interface is discussed in a number of blog posts and videos produced by COMSOL themselves:

  * [Introduction to the Elastic Waves, Time Explicit Interface](<https://www.comsol.com/blogs/introduction-to-the-elastic-waves-time-explicit-interface/>)
  * [Using the Discontinuous Galerkin Method to Model Linear Ultrasound](<https://www.comsol.com/blogs/using-the-discontinuous-galerkin-method-to-model-linear-ultrasound/>)
  * [Modeling Piezoelectricity Using the Discontinuous Galerkin Method](<https://www.comsol.com/blogs/modeling-piezoelectricity-using-the-discontinuous-galerkin-method/>)
  * [Simulating Elastic Waves in the Time Domain Using COMSOL®](<https://www.comsol.com/video/simulating-elastic-waves-in-the-time-domain-using-comsol>)

This method has a number of advantages over time implicit methods (Solid Mechanics) in terms of memory efficiency (the ultrasound flow meter model linked above consists of 7.6 million degrees of freedom (DoF) and can be solved using 9.5 GB of RAM), and the ability to use the "Absorbing Layers" feature in a time dependant study (similar to "Perfectly Matched Layers" in a frequency domain study), but in my experience these benefits are negated by the fact that the time stepping of the simulation is controlled by the smallest mesh element of the model. When dealing with complex geometry this results in a very slow solver.

I prefer to use the Solid Mechanics physics interface, but rather than using a Time Dependant study which uses a lot of RAM, use a Frequency Domain study followed by a Frequency to Time FFT step. This is both relatively fast to solve as well as being quite memory efficient. Another key benefit of this method over operating purely in the time domain is that Perfectly Matched Layers (PMLs) can only be used in the frequency domain, and so boundary reflections can now be effectively controlled. This is particularly important for ultrasonic guided wave simulations as low-reflecting boundary conditions do not effectively absorb anti-symmetric modes.

This method is briefly described in [an article by Ghose and Balasubramaniam](<https://www.ndt.net/article/apcndt2013/papers/294.pdf>), but the specifics of its implementation are not well documented. In this tutorial I will take you through the steps necessary to build a model using this principle. 

### Model geometry

For this example we will simulate the propagation of the A0 and S0 modes in a 2 mm thick Aluminium plate using a 3D model. 

![](/assets/img/wp-uploads/2023/08/geo.png)

The excitation point is on the left (y-axis point load) and the receiver point is on the right (y-axis displacement point probe). You can also use "Prescribed Displacement" at the excitation point but this doesn't allow the point to freely move after the excitation has taken place and can't then be used as a receiver in a pulse-echo configuration. Separate domains created using the "Layers" feature of a geometry block surround the inner plate and are used as PMLs. 

To determine excitation frequency we must first consult dispersion curves generated for the material in question. The group velocity curves below were generated using [The Dispersion Calculator](<https://www.dlr.de/zlp/en/desktopdefault.aspx/tabid-14332/24874_read-61142/>).

![](/assets/img/wp-uploads/2023/08/disp.png)Group velocity of 2 mm thick Aluminium.

We can see from these curves that operating at a frequency of 500 kHz will result in the excitation of both A0 and S0. The slowest velocity is that of A0, 3110 m/s, which will determine the wavelength used to generate the mesh.

### Excitation signal

As we are working in the frequency domain the excitation needs to be applied accordingly. This can be achieved by taking an FFT of the pulse that we want to apply in the time-domain, and bringing it into COMSOL as an interpolation function. I have generated a pulse in MATLAB and carried out the FFT there. 

For a time dependant study you can generate a Hamming windowed pulse in COMSOL using the following analytic expression:
    
    
```matlab
(sin(2*pi*f_0*t)*(t<(np/f_0)))*(0.54 - 0.46*(cos(2*pi*(t/(T0*np)))))
```

Where `f_0` is the frequency of excitation, `np` is the number of cycles, and `T0` is equal to `1/f_0`. These are all defined under global definitions. The argument is `t` and the function is `N`.

![](/assets/img/wp-uploads/2023/08/pulse-1.png)

To generate this same pulse in MATLAB and carry out an FFT you can use the following code:
    
    
```matlab
clear all

f_0=0.5e6; %frequency
np=5; %num cycles
fs = 1e8; %sample rate
T0=1/f_0;
T5 = 0:1/fs:5*T0;

% generate 5 cycle pulse
pulse = sin(2*pi*f_0*T5).*(T5<(np/f_0));

% generate hamming window
window = hamming(length(T5),'symmetric')';

%multiply together
ham_pulse = pulse.*window;

%pad array for FFT
padsize = 5000;
ham_pulse_pad = padarray(ham_pulse,, 0,'post'); % zero pad

% Perform FFT
N = length(ham_pulse_pad);
Y = fft(ham_pulse_pad);
P2 = abs(Y / N);
P1 = P2(1:N/2+1);
P1(2:end-1) = 2*P1(2:end-1);

% Define the frequency axis
f = fs*(0:(N/2))/N;

P1 = normalize(P1,"range");

% generate txt file for COMSOL
fft_array = transpose();
writematrix(fft_array ,'fft_pulse.txt','Delimiter','tab')

%% plot
figure
tiledlayout(1, 2);
nexttile
set(gca, 'LineWidth', 1.2); % For grid lines
hold on
grid on 
box on
set(gca, 'fontsize', 16);
plot(T5,ham_pulse, 'LineWidth', 3)
ylabel('Amplitude (N)')
xlabel('Time (μs)')

nexttile
set(gca, 'LineWidth', 1.2); % For grid lines
hold on
grid on 
box on
set(gca, 'fontsize', 16);
plot(f, P1, 'LineWidth', 3);
xlim()
ylabel('Amplitude')
xlabel('Frequency (Hz)')

x0=10;
y0=10;
width=2000;
height=1000;
set(gcf,'position',)
```

![](/assets/img/wp-uploads/2023/08/matlab-1.png)

This will produce a .txt file containing a FFT of the Hamming windowed pulse that can be imported into COMSOL using an interpolation function.

[fft_pulse_500kHz](<https://lawrenceyule.com/wp-content/uploads/2023/08/fft_pulse_500kHz.txt>)[Download](<https://lawrenceyule.com/wp-content/uploads/2023/08/fft_pulse_500kHz.txt>)

##### two-sided excitation

You can control the excitation of either symmetric or anti-symmetric modes by using two-sided excitation. Place a second point load on the lower boundary directly below the first point. Apply the input signals in phase to selectively excite only the anti-symmetric modes. Apply one signal 180 degrees out of phase to selectively excite only the symmetric modes. 

![](/assets/img/wp-uploads/2023/08/two-sided.png)Two-sided excitation response at receiver position. S0 in green, A0 in blue.

You can use a parametric sweep to run the simulation for both mode types. First define a global parameter of `mode` with a value of 1 or -1. Then set the second point load to `int1(freq)*mode`. Now create a `parametric sweep` under `Study 1`, add `mode` as a parameter, and use values of `1 -1`.

### Meshing

The number of mesh elements in your model should be determined by the wavelength of the slowest propagating wave mode in that material. The maximum element size should be set to `velocity/N/f_0` where `N` is 5 or 6 elements per wavelength. This is the sweet spot of accurately representing waves without using an excessive number of elements. AltaSimTechnologies have created [an excellent video](<https://www.youtube.com/watch?v=yWtazWjH5nQ>) to demonstrate this. Note that in our model we are using quadratic discretisation (the default) rather than linear discretisation, which halves the number of elements required (10-12 > 5-6).

In terms of the type of mesh to use, I have found that using a mapped/swept mesh wherever possible is the best way to reduce the number of mesh elements used, but this is only effective for quite simple geometries. A compromise can be achieved by combining a free triangular boundary generator with a swept mesh.

![](/assets/img/wp-uploads/2023/08/mesh-settings.png)

![](/assets/img/wp-uploads/2023/08/mesh-1024x400.png)

Use a "Distribution" node to ensure that there are at least 2 elements across thin regions. Perfectly Matched Layers (PML) should be meshed using a swept mesh, with 5-8 elements across the thickness.

See the following articles for more information:

  * [Performing a Mesh Refinement Study](<https://www.comsol.com/support/knowledgebase/1261>)
  * [Best Practices for Meshing Domains with Different Size Settings](<https://www.comsol.com/blogs/best-practices-for-meshing-domains-with-different-size-settings/>)
  * [How to Inspect Your Mesh in COMSOL Multiphysics®](<https://www.comsol.com/blogs/how-to-inspect-your-mesh-in-comsol-multiphysics/>)
  * [Geometry and Mesh Setup for Modeling Regions of Infinite Extent](<https://www.comsol.com/support/learning-center/article/Geometry-and-Mesh-Setup-for-Modeling-Regions-of-Infinite-Extent-52791>)

### Study steps

Here I will describe the differences between using a time dependant study and using a two-step frequency domain study. A model can easily be adapted to use either method.

In both cases a suitable time step should be defined as a parameter under global definitions. The equation for this is [discussed by COMSOL here](<https://www.comsol.com/support/knowledgebase/1118>). Our `time_step` is determined by the formula: `CFL/(N*f_max)` where the CFL number is `0.1`, `N` is `5`, and `f_max` is equal to `1.4*f_0`.

##### Time dependant study

Set `output times` under `Study 1>Step 1: Time Dependent>Study Settings` to a range from `0` to `sim_length`, in increments of `time_step`.

The `steps taken by solver` under `Solver Configurations>Solution 1>Time-Dependant Solver 1>Time Stepping` should be set to `Manual` and the `Time step` should be set to `time_step` as defined under global definitions.

![](/assets/img/wp-uploads/2023/08/time_dep.png)

Ensure that the point load function is set to use the time domain excitation signal `an1(t)`.

##### Frequency domain study

The simulation is carried out in two study steps. Firstly a "frequency domain" study, followed by a "frequency to time FFT" study. In the first step the frequency range of the study is set to cover the bandwidth of the pulse, which can be determined by looking at the FFT calculated earlier. In this case the range bandwidth is 200-900 kHz. The frequency step determines the length of the time domain simulation, so it should be adjusted according to your propagation distance to ensure that the whole signal is sufficiently represented. I have used 20 kHz in this example.

![](/assets/img/wp-uploads/2023/08/freq_study.png)

![](/assets/img/wp-uploads/2023/08/freq_study_1.png)

Set `output times` under `Study 1>Step 2: Frequency to Time FFT>Study Settings` to a range from `0` to `sim_length`, in increments of `time_step`.

Ensure that the point load function is set to use the frequency domain excitation signal `int1(freq)`.

##### optimisation

If you are only interested in the response at your probes you can reduce the amount of data stored in your model by creating a probe selection under `definitions` and then selecting that within `Step 1>Values of Dependent Variables>Store fields in output`. This applies to both simulation methods. If you want to visualise propagation consider selecting only a single boundary (e.g the top face).

![](/assets/img/wp-uploads/2023/08/probes.png)

## Results

![](/assets/img/wp-uploads/2023/08/A0S0-800kHz-2D.png)Displacement with exaggerated deformation in a 2D simulation. A0 and S0 clearly visible.

Here the slower A0 mode can be seen on the left, exhibiting substantial out-of-plane motion as you would expect. The faster S0 mode can be seen on the right, where only in-plane motion can be seen.

![](/assets/img/wp-uploads/2023/08/signal-1.png)Displacement (y-axis) response at receiver position.

At the receiver position the two modes are separated in time, with the lower amplitude S0 mode arriving first. 

You can download the model below:

[freq_alu_500kHz_3D](<https://lawrenceyule.com/wp-content/uploads/2023/08/freq_alu_500kHz_3D.zip>)[Download](<https://lawrenceyule.com/wp-content/uploads/2023/08/freq_alu_500kHz_3D.zip>)

## Time vs Frequency domain

Operating at a frequency of 500 kHz with 122,091 degrees of freedom solved for, the solution times of the same model are:

**Frequency domain:** 2m 41s - 9 GB RAM  
**Time domain:** 5m 9s - 4GB RAM

The frequency domain model solves substantially faster than the time domain model, although the RAM usage is higher. For this small and simple model the RAM required for the Frequency to Time FFT step exceeds that used for the frequency domain step. For a larger model, for example that of a printed circuit board with 1,675,626 DoF, the first step requires 101 GB of RAM while the FFT step only requires 25 GB. Attempting to solve this model with a time dependant study would exceed the 193 GB of RAM available on this compute cluster.

![](/assets/img/wp-uploads/2023/08/PCB.png)Printed circuit board model.

The difference between methods is compounded by the inability to fully absorb boundary reflections in the time domain simulation, and so the geometry must be extended to sufficiently separate reflections from the direct signal.

There are some caveats to this method of course, namely that we are limiting the frequency bandwidth of the model to that of the excitation signal. This is appropriate to certain guided wave applications but not necessarily all ultrasonic applications.

In future posts I will discuss the implementation of piezoelectric sensors, as well as heat transfer simulations. 

Do you have any tips for ultrasound simulations in COMSOL? Please leave a comment!
