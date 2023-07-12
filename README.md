# HistoScanner - An open-source slidescanning system for histological slides using ImSwitch and ASHLAR

Welcome to the openOCT Hackathon! In this hackathon, we will be addressing the crucial need for an open-source slide scanner that can help capturing large area microscopic samples for example for digital pathology. We will provide an openUC2-based XYZ stage scanning microscope that can be hooked up to the ImSwitch microscopy control software. The challenge is to automate the acquisition process and buildup a pipeline that adapts the data to the stitching software ASHLAR (https://github.com/labsyspharm/ashlar/)



![](IMAGES/WhatsApp Image 2023-07-07 at 10.27.11.jpeg)

# Open-source Slide Scanner Documentation

This documentation provides an overview of the open-source slide scanner project, which aims to develop a low-cost, fast, and open solution for large area scanning of microscopic samples. The motivation behind the project is to address the labor-intensive and expensive nature of manual tissue analysis in microscopy and the limited accessibility of commercially available slide scanners. The goal is to create a tool that can be integrated into existing workflows and interact with open-source data analysis pipelines.

## Motivation

Microscopy plays a crucial role in identifying cancer and studying tissue samples. Various labeling strategies, such as HeMa and antibody labeling, allow for different aspects of cells and tissues to be visualized. However, manual analysis is time-consuming and requires significant labor. Commercially available slide scanners are often expensive and proprietary, making it challenging to extract data and integrate them into existing workflows. There is a need for a low-cost, open-source slide scanner that can perform large area scanning quickly and efficiently. The captured data should be compatible with open-source data analysis pipelines like ASHLAR for stitching and QPath for annotation and analysis.

## Goals

The primary goals of the open-source slide scanner project are as follows:

### 1. Proper Relation between Physical and Digital Coordinates

Ensure a precise and accurate mapping between physical coordinates (sample plane) and digital coordinates (pixel location, XYZ steps for motors). Calibration of dimensions should ideally be performed autonomously.

### 2. Stitching of Image Tiles

Develop a working end-to-end solution for stitching image tiles into an `.ome.tiff` or equivalent file format. This will enable the generation of a complete and seamless representation of the scanned slide.

### 3. Brightfield Imaging

Implement brightfield imaging using an RGB camera. This will allow for the visualization of samples in natural colors.

### 4. Region of Interest Definition

Enable the definition of a region of interest to be scanned. Users should be able to define a bounding box or specific areas on the slide to focus on during scanning.

### 5. Pre-Scanning and Time Estimation

Incorporate a pre-scanning feature to capture low-resolution thumbnails of the slide for preview purposes. Additionally, provide an estimated time for the total slide scan based on the defined region of interest.

### 6. Accuracy Measurement

Measure the approximate error of the scanner in relation to the indicated XY positions. The error should be below 300um to ensure accurate stitching of multiplexed images in ASHLAR.

### 7. Automatic and Manual Focus Adjustment

Develop mechanisms for automatic and manual focus adjustments to ensure clear and precise imaging at different depths within the sample.

Secondary goals include achieving lossless image generation, implementing fluorescence imaging capabilities, integrating the hardware and software into Opentrons2 robotic platforms, and incorporating barcode label scanning.

## Current State: Building the Prototype

The project is currently focused on building the prototype of the slide scanner. The UC2-based microscope, named "Jordiscope," serves as the foundation for this project. The microscope features XYZ-stage scanning with micrometer precision, an RGB camera (HIK Vision), a 20x objective lens with 0.45NA, LED illumination in Köhler configuration, and a motorized focus mechanism.

The prototype development will involve integrating the microscope with additional components and developing the necessary software to achieve the project's goals. The team will continue working on refining the hardware and software to create a functional and efficient open-source slide scanner.

For more detailed information, including instructions, schematics, and code implementation, please refer to the project's GitHub repository. The project welcomes collaboration and contributions from the community to collectively build an accessible and powerful tool for microscopy analysis.


#### ImSwitch

We want to be able to use this as an online-running widget that includes the goals from above. For this we can base our work on the HistoScan Widget and Controller in ImSwitch (UC2). Relevant code passages are

*The Widget*
https://github.com/openUC2/ImSwitch/blob/master/imswitch/imcontrol/view/widgets/HistoScanWidget.py

*The Controller*
https://github.com/openUC2/ImSwitch/blob/master/imswitch/imcontrol/controller/controllers/StandaPositionerController.py

*The Manager*
https://github.com/openUC2/ImSwitch/blob/master/imswitch/imcontrol/model/managers/HistoScanManager.py

*The ESP32-based camera* (for the coarse prescan from above)
https://github.com/openUC2/ImSwitch/blob/d3009b2dd669fbee273cca770c87c8c80d0c5940/imswitch/imcontrol/model/interfaces/CameraESP32CamSerial.py#L37


The general mockup looks like this:
![](IMAGES/Bildschirmaufnahme 2023-07-08 um 10.55.02.gif)



#### Setting up ImSwitch

You can follow the following steps described here:
https://github.com/openUC2/ImSwitch#option-c-install-from-github-uc2-version

Please install Anaconda or miniconda previously.

We also provide an experimental installer that installs the serial drivers for the CP20XX serial UART interface: https://github.com/beniroquai/ImSwitchInstaller/tree/main/constructor

(adapt for windows accordingly)
```
cd ~/Documents
git clone https://github.com/openUC2/ImSwitch/
cd ImSwitch
# alternatively download this repo, unzip the .zip-file and open the command prompt in this directory
conda create -n imswitch python=3.9 -y
conda activate imswitch
pip install -e ./ --user
# optional:
pip install git+https://gitlab.com/bionanoimaging/nanoimagingpack
```

Then you would need the proper config file. You can find more information here: https://github.com/beniroquai/ImSwitchConfig

Briefly:
```
cd ~/Documents
# if there is a folder called ImSwitchConfig => rename it!
git clone https://github.com/beniroquai/ImSwitchConfig
# Alternatively download the repository as a zip, unzip the file into the folder Documents/ImSwitchConfig
```

Then inside the config file speficy the `.json` file that describes your setup and add `HistoScan` to the available modules

#### Potentially working OME.TIFF implementation for ASHLAR


```py
        #%% imports
        from pathlib import Path
        import numpy as np
        import tifffile as tif
        from datetime import datetime

        # Define metadata
        metadata = {
            'axes': 'TZCYX',
            'Image': {
                '@ID': 'Image:8',
                '@Name': 'COLNOR69MW2@20191105_163047_059221.rcpnl #9',
                'AcquisitionDate': datetime(2019, 11, 5, 16, 56, 27).isoformat(),
                'InstrumentRef': {'@ID': 'Instrument:0'},
                'ObjectiveSettings': {'@ID': 'Objective:18108'},
                'Pixels': {
                    '@BigEndian': 'false',
                    '@DimensionOrder': 'XYCZT',
                    '@ID': 'Pixels:8',
                    '@Interleaved': 'false',
                    '@PhysicalSizeX': '0.6499999761581421',
                    '@PhysicalSizeXUnit': 'µm',
                    '@PhysicalSizeY': '0.6499999761581421',
                    '@PhysicalSizeYUnit': 'µm',
                    '@PhysicalSizeZ': '0.7662222385406494',
                    '@PhysicalSizeZUnit': 'µm',
                    '@SignificantBits': '16',
                    '@SizeC': '1',
                    '@SizeT': '1',
                    '@SizeX': '1280',
                    '@SizeY': '1080',
                    '@SizeZ': '1',
                    '@Type': 'uint16',
                    'Channel': [
                        {'@ID': 'Channel:8:0', '@EmissionWavelength': '431.0', '@EmissionWavelengthUnit': 'nm', '@ExcitationWavelength': '395.0', '@ExcitationWavelengthUnit': 'nm', '@NDFilter': '0.0', '@SamplesPerPixel': '1', 'LightPath': None},
                        {'@ID': 'Channel:8:1', '@EmissionWavelength': '525.0', '@EmissionWavelengthUnit': 'nm', '@ExcitationWavelength': '485.0', '@ExcitationWavelengthUnit': 'nm', '@NDFilter': '0.0', '@SamplesPerPixel': '1', 'LightPath': None},
                        {'@ID': 'Channel:8:2', '@EmissionWavelength': '590.0', '@EmissionWavelengthUnit': 'nm', '@ExcitationWavelength': '555.0', '@ExcitationWavelengthUnit': 'nm', '@NDFilter': '0.0', '@SamplesPerPixel': '1', 'LightPath': None}
                    ]}}}

        def create_tiff(filename, shape, dtype):
            # create an empty OME-TIFF file the size of the scan required
            tif.imwrite(filename,
                    shape=shape,
                    dtype=dtype,
                    metadata=metadata)


        def write_memmapped_tiff(filename, data, idx, dtype):
            # incoming image is a list if called from LabVIEW. Convert to numpy array first

            if isinstance(data, list):
                np_image = np.array(data, dtype=dtype)
            else:
                np_image = data

            image_stack = tif.memmap(filename)
            image_stack[idx] = data
            image_stack.flush()



        # create a random z-stack image similar to the Zeiss LSM 510 12 bit tiff
        # One region, consisting of 25 z-slices, 2 channels, x*y of 512*512 pixels

        fpath = Path("multi_stack_img1.ome.tiff")
        # TZCYX
        shape = (10, 25, 2, 128, 128)
        dtype = 'uint16'

        # create a new file and write data to it
        create_tiff(fpath, shape, dtype)

        # write slices to memory-mapped tiff file
        for region in range(shape[0]):
            write_memmapped_tiff(fpath, np.random.randint(0, 2 ** 12, shape[1:], dtype=dtype), region, dtype)
```

## Safety

Don't hurt yourself!

## Challenge: Enhancements for the Open-source Slide Scanner

The open-source slide scanner project faces several challenges that need to be addressed for improved functionality and performance. Here are the main challenges along with proposed enhancements:

1. Familiarity with Image Acquisition and REST API: Start by becoming familiar with image acquisition using the built-in REST API. This will allow you to understand the image acquisition process and explore the capabilities of the slide scanner. Utilize the provided API documentation to learn how to control and interact with the scanner.

2. Development of ImSwitch Plugin for Scanning: Create an ImSwitch Plugin that performs scanning based on a predefined list of positions. This plugin will streamline the scanning process by automatically moving the slide to the specified positions and acquiring images. It should integrate with the existing ImSwitch software and provide a user-friendly interface for defining the scanning positions.

3. Integration of Pre-scan Camera and Mapping of Sample Coordinates: Enhance the slide scanner by incorporating a pre-scan camera. This camera will capture low-resolution images of the slide, allowing users to preview the sample and define the regions of interest. Develop a mapping mechanism that correlates the sample coordinates with the scanning positions, ensuring accurate image acquisition.

4. Data Storage Organization for ASHLAR Compatibility: Organize the acquired image data in a structured format that can be easily read by ASHLAR, an existing data analysis pipeline. Ensure that the image data is stored in a compatible format, including metadata such as coordinates, timestamps, and any other relevant information required by ASHLAR for stitching and analysis.

5. Pathological Slide Scanning and Stitching: Perform scanning on a pathological slide to test the slide scanner's capabilities. Acquire images at various positions and use ASHLAR or other stitching methods to combine the images into a complete and seamless representation of the slide. Evaluate the quality and accuracy of the stitching process.

6. Comparison: Compare the performance and usability of the ImSwitch Plugin with the REST API. Evaluate the advantages and disadvantages of each method in terms of speed, flexibility, and user experience. Collect feedback from users and identify areas for improvement.

7. Calibration of Pixel, Motion, and Resolution: Calibrate the relationship between pixel dimensions, motor movements, and resolution. Ensure accurate mapping between physical coordinates and digital image coordinates. Develop calibration routines or algorithms to establish the correct scaling and alignment of the scanned images.

8. Integration of a Coarse Scanning Camera: Explore the possibility of integrating a secondary camera specifically designed for coarse scanning. This camera can capture low-resolution images over larger areas, allowing for faster scanning while still maintaining accuracy. Develop software and hardware integration to synchronize the coarse scanning camera with the slide scanner.

9. Full Scan and Adaptive Grid: Implement a full scan capability that covers the entire slide area. This can be achieved using an adaptive grid-based approach, where the scanning positions are dynamically determined based on the sample size and slide dimensions. Develop algorithms and automation to optimize the scanning process and minimize overlap between tiles.

10. Real-time Stitching with Napari Integration: Enhance the slide scanner software to support real-time stitching and visualization using tools like Napari. Implement online stitching algorithms that can combine image tiles as they are acquired, providing immediate feedback to users. Integrate the real-time stitched image display into the user interface for seamless operation.

11. Feedback and Quality Assessment: Encourage users to provide feedback on the features and quality of the slide scanner. Collect user input to identify areas for improvement, such as hardware components, software functionality, or overall user experience. Use this feedback to iteratively enhance the slide scanner and address any limitations or issues.

12. Consideration of OpenFlexure's ASHLAR: Explore the possibility of integrating OpenFlexure's ASHLAR, a stitching pipeline, into the slide scanner project. Assess the advantages and features of ASHLAR, such as its re-jigging images without rerunning feature, auto thresholds, and optimization for various hardware conditions. Evaluate the compatibility and potential benefits of incorporating ASHLAR into the slide scanner workflow.

By addressing these challenges and implementing the proposed enhancements, the open-source slide scanner will become a powerful and versatile tool for fast and accurate scanning of microscopic samples.

## Ressources



## License
