# Contrast Detection Probability Calculator


This repository provides:

* A toolbox for calculating [contrast detection probability](https://www.image-engineering.de/content/library/conference_papers/2019_02_28/EIC2019_CDP.pdf) (CDP) of an imaging system

* A simulated imaging system for generating HDR images given scene luminance map, and alternatively

* A utility for annotating and extracting RoIs from CDP test patterns in [Image Engineering<sup>®</sup> DTS](https://www.image-engineering.de/products/solutions/all-solutions/dynamic-test-stand) device


# What is CDP?

Contrast detection probability (CDP) is a metric to *describe the performance of an imaging system to reproduce contrasts in the physical scenes*. Since a distinguishable contrast in an image is a prerequisite for modern computer vision algorithms, CDP can be regarded as a core functionality to measure the *CV-oriented image quality* of an imaging system.

More detailed introduction and use cases about CDP can be found in [\[1\]](#ref1), [\[2\]](#ref2), and [\[3\]](#ref3).

# Basic idea

By its theoretical definition, CDP is the ratio of the area under a contrast-to-pdf (probability distribution function) curve, constrained by an upper and a lower bounds (<img src="https://render.githubusercontent.com/render/math?math=C_{in}\pm\epsilon">, where <img src="https://render.githubusercontent.com/render/math?math=C_{in}"> is the target contrast and <img src="https://render.githubusercontent.com/render/math?math=\epsilon"> is a confidence interval), to the whole area under this pdf curve (see figure below). In a discrete digital imaging system, however, we can only obtain this ratio by dividing the number of observations withing the bounds by the number of all observations.


<p align="center">
<img src="figures/cdp.png" width="640px"/></p>

<p align="center"><em>CDP definition in <a href="#ref2">[2]</a></em><p align="center">


Inspired by Image Engineering<sup>®</sup> [DTS-Evaluation software](https://www.image-engineering.de/content/products/solutions/dynamic_test_stand/downloads/Dynamic_Test_Stand_product_summary.pdf), our toolbox calculates CDP based on an RoI-to-RoI manner (see figure below), i.e., given two flatten patches with known contrast and luminance levels (in cd/m<sup>2</sup>), the CDP is calculated by counting the number of pixel-pairs from two image RoIs that are able to reproduce that contrast value, then dividing by the number of all possible pixel-pairs.


<p align="center">
<img src="figures/roi-to-roi.png" width="480px"/></p>

<p align="center"><em>We calculate CDP based on an RoI-to-RoI manner. The figure is copied from <a href="#ref5">[5]</a>, and more appropriately, the y-label should be "the normalized frequency"</em><p align="center">


For example, given two neutral patches in the physical world with luminance 100cd/m<sup>2</sup> and 300cd/m<sup>2</sup> respectively, their contrast is <img src="https://render.githubusercontent.com/render/math?math=\frac{300-100}{300%2B100}=0.5"> following the Michelson contrast definition. Suppose we use a camera to capture this scene, and get an output image with two 10*10 square RoIs corresponding to these two patches, then the CDP can be obtained by counting the ratio of all pixel-pairs that reproduce the target contrast (based on pixel values), to the total 100<sup>2</sup>=10,000 pixel-pairs.

A CDP value is a function of the target contrast value, the target luminance level (by averaging the luminance of two patches), and, of course, the characteristic of the imaging system. So if camera A produces a higher CDP values then camera B, we can say that camera A has higher probability to distinguish the objects with a certain degree of difference, and at a certain luminance level, in their real use scenarios.

# Usage

### Demo

A comprehensive demo was provided, simply by running 

```bash
python demo.py
``` 

and annotating the CDP charts following the instructions, you will get results on the `./simulation/simulated_images` directory.


### CDP Calculator

The [`CDPCalculator`](cdp_calculator.py#L12) class now supports two calculating modes: *user-specified target contrast(s) mode* and *all target contrasts mode*:

* The user-specified target contrast(s) mode is run by calling [`calculate()`](cdp_calculator.py#L43) method. Given a set of patches with known luminance levels and an interesting target contrast value (or several contrast values), this method will search from all patch-pairs and find those that satisfy this target contrast value(s). The CDP values will be calculated for all these patch-pairs. See figure below.  


<p align="center">
<img src="figures/cdp_contrasts.png" width="580px"/></p>

<p align="center"><em>CDPs with respect to different luminance levels. Here four target contrasts (6%, 10%, 20%, and 30%) are evaluated. Two regions with CDP drops in the middle can be clearly observed because there exists <a href="https://gvv.mpi-inf.mpg.de/projects/opthdr/granados10_opthdr.pdf">SNR drop</a> during HDR blending. The right side drop is caused by the response saturation</em><p align="center">


* The all target contrasts mode is run by calling [`calculate_all()`](cdp_calculator.py#L93) method. In this mode the CDP values will be calculated for all possible patch-pairs. See figures below.  


<p align="center"><img src="figures/cdp_all.png" width="720px"/></p>

<p align="center"><em>CDPs with respect to different target contrasts and luminance levels</em><p align="center">


<p align="center"><img src="figures/cdp_all_2d.png" width="580px"/></p>

<p align="center"><em>Same as above figure, in top view</em><p align="center">

See the documents in the [cdp_calculator.py](cdp_calculator.py) for more info about its usage.


### Simulated Imaging System & DTS RoIs Extractor

If you do not own a DTS device, this toolbox provides a [simulated CDP pattern generator](simulation/simulated_patterns.py#L36) and a [simulated imaging system](simulation/simulated_camera.py#L7) to help generate synthetic images. See figure below.


<p align="center"><img src="figures/simulated_image.png" width="540px"/></p>

<p align="center"><em>Synthetic HDR image of CDP pattern generated by <a href="simulation/simulated_camera.py#L45">capture_hdr()</a></em><p align="center">


A convenient utility is also provided for users to annotate and extract RoIs from real or synthetic CDP images. See figures below.


<p align="center"><img src="figures/rois_extractor.png" width="600px"/></p>

<p align="center"><em>RoIs extractor</em><p align="center">


<p align="center"><img src="figures/rois.png" width="600px"/></p>

<p align="center"><em>Extracted RoIs</em><p align="center">


# Reference

<a name="ref1">1.</a> Detection Probabilities: Performance Prediction for Sensors of Autonomous Vehicles. Geese M, Seger U, and Paolillo A (2018)

<a name="ref2">2.</a> Contrast Detection Probability - Implementation and Use Cases. Artmann U, Geese M, Gäde M (2019)

<a name="ref3">3.</a> Imatest Documentation: Contrast Detection Probability. https://www.imatest.com/docs/cdp/

<a name="ref4">4.</a> ISO 12232:2019 Photography — Digital still cameras — Determination of exposure index, ISO speed ratings, standard output sensitivity, and recommended exposure index

<a name="ref5">5.</a> Comparison of Detectability Index and Contrast Detection
Probability. Robin B. Jenkin (2019)

<a name="ref6">6.</a> Implementierung von CDP: Entwicklung eines Programmiercodes in Python zur Untersuchung und Messung von CDP bei Fahrerassistenzkameras. Lukas Ebbert (2018)



# License

Copyright 2020 Qiu Jueqin

Licensed under [MIT](http://opensource.org/licenses/MIT).