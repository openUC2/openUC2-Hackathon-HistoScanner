# HistoScanner - An open-source slidescanning system for histological slides using ImSwitch and ASHLAR

Welcome to the openOCT Hackathon! In this hackathon, we will be addressing the crucial need for an open-source optical coherence tomography system that can help students learning more about setting up a low-coherence interferemoter. To keep the budget low, we aim for a full-field time domain OCT using an SLD (super luminescent diode) - alternatively one could also try an LED to lower the costs even more


Primary Goals
- Working end to end solution to stitch image tiles into an .ome.tiff (or equivalent)
- Brightfield imaging
- Ability to define a region of interest to be scanned (for example, defining a bounding box)
- "Pre-scanning" (low resolution thumbnail)
- Time estimate for total slide scan
- Measure the approximate error of the scanner in relation with the indicated xy positions (should be <300um for stitching multiplexed images in Ashlar
- Automatic & Manual Focus setting adjustments

Secondary Goals
- Lossless image generation
- Fluorescence imaging
- Hardware and Software Integration into Opentrons2
- barcode label scanning



ImSwitch Plugin vs REST API




## Motivation


## Goal


## Background


## Current State: Building the Prototype


### Setting up and Adapting

### Optical Setup


### test Setup

### Code


#### ESP32


#### PYTHON

#### Image Processing


#### ImSwitch

We want to be able to use this as an online-running autofocus in ImSwitch. Relevant code passages are the camera interface for the herein created USB-serial camera:

https://github.com/openUC2/ImSwitch/blob/d3009b2dd669fbee273cca770c87c8c80d0c5940/imswitch/imcontrol/model/interfaces/CameraESP32CamSerial.py#L37

The FocusLockController

https://github.com/openUC2/ImSwitch/blob/d3009b2dd669fbee273cca770c87c8c80d0c5940/imswitch/imcontrol/controller/controllers/FocusLockController.py#L26

### How to set up?

## Safety

## Challenge

### Varitions

## When Goal is met?

- have one working system
- Detect the focus as one number from the astimatism as a function of the z-position
- have a controller that may give step-values for the focus motor
- have a functioning focuscontroller inside imswitch that can keep the focus

## Ressources
- https://hackaday.com/2021/02/02/dvd-optics-power-this-scanning-laser-microscope/


## License
