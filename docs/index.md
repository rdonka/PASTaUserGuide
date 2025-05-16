# **PASTa**: **P**hotometry **A**nalysis and **S**ignal Processing **T**oolbox

![png](./img/home_1_PASTalogo.png)

Welcome to the documentation site for __PASTa__ (**P**hotometry **A**nalysis and **S**ignal Processing **T**oolbox)! The PASTa protocol is an open source toolbox and protocol for the preparation, signal processing, and analysis of fiber photometry data. 

Fiber photometry is a rapidly growing technique to record real-time neural signaling in awake, behaving subjects. However, the processing and analysis of photometry data streams can be complicated, and there is wide divergence in methods across the field. While several open-source signal processing tools exist, platforms can be inflexible in accommodating experimental designs, lack consistency in signal peak detection, and be difficult to use for naive users.

To remedy these challenges, we developed PASTa, an open-source MATLAB based toolbox and protocol for the processing and analysis of fiber photometry data. PASTa includes a full analysis pipeline from data preparation through signal processing and transient event detection. Default parameters were selected to provide users with a conservative starting place, with optional inputs to include other commonly used methods and techniques in the field. Additionally, the transient detection protocol adopts options for determining peak detection thresholds and pre-peak baselines to allow more reliable detection of events and characterization of transient kinetics. The pipeline is designed to process multiple sessions at once, automating data processing, analysis, and plot creation to ensure application consistency across experimental sessions.

While operating through MATLAB, the code is annotated to be readable, accessible for new users, and adaptable. Here you'll find a detailed user guide for each stage of PASTa, including installation instructions, detailed guidelines for each stage of the signal processing and analysis pipeline, example analyses, and additional details on function inputs and usage to support the documenation available within each function. 

![png](./img/home_2_PASTaprotocoloverview.png)


## Navigating the PASTa Repository and Documentation
The PASTa code is openly available on [GitHub](https://github.com/rdonka/PASTa). The toolbox is also available as a toolbox through [MATLAB File Exchange](https://www.mathworks.com/matlabcentral/fileexchange/181125-pasta). This user guide contains detailed documentation, instructions for function use, and troubleshooting tips. 

### Getting Started
The [Getting Started](https://rdonka.github.io/PASTaUserGuide/gettingstarted/) page includes quick start instructions and installation tips. This is a great place to start if you haven't previously downloaded or implemented the PASTa Protocol on your device.

### User Guide
The [User Guide](https://rdonka.github.io/PASTaUserGuide/userguide/userguide/) page includes detailed instructions and examples for every step of the protocol, including [Data Preparation](https://rdonka.github.io/PASTaUserGuide/userguide/datapreparation/), [Signal Processing](https://rdonka.github.io/PASTaUserGuide/userguide/signalprocessing/), and [Transient Analysis](https://rdonka.github.io/PASTaUserGuide/userguide/transientanalysis/). This is the best place to start for new users. Each page will walk you through each step, with information about the reccomended settings and any defaults built in to the protocol. The guide also includes examples of each step with fiber photometry recordings with three sensors capturing mesolimbic dopamine dynamics to help users gain an intuition for what each phase of the process is doing to the data streams.

### Function Documentation
For using and troubleshooting individual functions, the [Function Documentation](https://rdonka.github.io/PASTaUserGuide/functiondocumentation/) page includes even more details about the use of each function in the toolbox, including notes and suggestions for troubleshooting errors. Detailed documentation is also commented in to each function in the source code - just view the help window or open the function source file in MATLAB to read.

### Example Analyses
Additionally, to provide an example of the full pipeline, the [Example Analyses](https://rdonka.github.io/PASTaUserGuide/exampleanalyses/) page provides users with full examples of experimental analyses. All files needed to perform the example analyses are available for download so users can walk through each step in MATLAB on their own devices to ensure that installation and set up were sucessful, and view outputs of a validated script.

# Contact Us!
Please feel free to reach out with questions, feedback, and feature requests. PASTa is actively maintained, and will continue to integrate new features into future versions. Version releases will be continually added to GitHub, and details are also available on the [Version Release Notes](https://rdonka.github.io/PASTaUserGuide/about/releasenotes/) page.

To facilitate community discussion, we are actively maintaining the [Discussions Forum](https://github.com/rdonka/PASTa/discussions) via GitHub, which includes channels for announcements, feature requests, troubleshooting, and general discussions. Our individual contact information is also available [here](https://rdonka.github.io/PASTaUserGuide/contactus/).
