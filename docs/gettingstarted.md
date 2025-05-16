# Getting Started with PASTa
Here you'll find the quick start guide to set up __PASTa__ with instructions on installation and basic use of the toolbox. Once your device is set up, move to the [User Guide](https://rdonka.github.io/PASTaUserGuide/userguide/userguide/) for detailed documentation on the use of the PASTa Protocol for analysis.

# Set Up and Installation
PASTa is an open-source MATLAB-based toolbox, containing a series of functions to extract, process, and analyze photometry data. Users must first install MATLAB (version 2020a or later) and add the Signal Processing Toolbox add-on. Following MATLAB installation, users can install PASTa three ways: __1)__ MATLAB Add-On Explorer, __2)__ downloading the PASTa MATLAB toolbox installation file from the PASTa GitHub, or __3)__ local installation of the PASTa GitHub repository.

## Install MATLAB
First, users must ensure that MATLAB is installed on the local computer used for analysis. The _Signal Processing Toolbox_ is a required dependency of PASTa.

To install MATLAB, visit the [MathWorks site](https://www.mathworks.com/pricing-licensing.html). Multiple license types are available, including Academic ($275/year) and student ($99/year). For more info, see [MATLAB Pricing](https://www.mathworks.com/pricing-licensing.html). Licensing may also be available through your university.

After installing MATLAB, ensure the _Signal Processing Toolbox_ is installed as an Add-On. To install the Add-On:

- At the top of the MATLAB window, select the __Home__ tab

- Under __Environment__, select __Add-Ons__. Click _Get Add-Ons_

- In the __Search__ bar, search for _"Signal Processing Toolbox"_

- Install the _Signal Processing Toolbox_ Add-On

## Install PASTa

### 1) MATLAB Add-On Explorer
PASTa installation as a MATLAB toolbox through the MATLAB Add-On Explorer is the recommended method to install the toolbox to ensure MATLAB is set up to properly access the functions. This guarantees version stability and accessible for newer users of MATLAB as functions will be installed and accessed similarly to built-in MATLAB functions. 

To install PASTa through the MATLAB Add-On Explorer:

- At the top of the MATLAB window, select the __Home__ tab

- Under __Environment__, select __Add-Ons__. Click _Get Add-Ons_

- In the __Search__ bar, search for _"PASTa"_

- Install the _PASTa_ toolbox Add-On

### 2) MATLAB Toolbox Installation File on GitHub
Installation via the MATLAB toolbox file in the PASTa GitHub is another option and may be necessary for MATLAB users with older versions that are not compatible with the Add-On explorer or are not tied to a current MATLAB license subscription. 

To install PASTa through the MATLAB Toolbox file on GitHub:

- Navigate to the PASTa GitHub repository: https://github.com/rdonka/PASTa

- Download the file _"PASTa.mltbx"_

- Open the file _"PASTa.mltbx"_. The toolbox will be automatically installed to MATLAB.

### 3) Clone the PASTa GitHub Repository
To access PASTa, users can also copy the toolbox functions to a local folder of the user's choice and then add that folder to the MATLAB path. If desired, users can create a local copy of PASTa by cloning the repository via GitHub desktop. The use of GitHub desktop allows users to easily fetch updates to the repository. Installation of PASTa through cloning of the GitHub repository is more dynamic and may facilitate integration of version updates as they are released. New versions of PASTa will continue to include integration of newly developed signal processing methods, bug fixes, and analysis features as they are released.

To download and use PASTa directly via the GitHub repository:

- Download __GitHub__ and __GitHub Desktop__.

    - For detailed instructions on GitHub setup, see https://docs.github.com/en/desktop/installing-and-authenticating-to-github-desktop/setting-up-github-desktop

- Clone the PASTa repository.

    - To make it easy to locate repositories, we recommend making a folder on your desktop called "GitHubRepositories" and cloning all repositories there. 

    - For detailed instructions on cloning repositories, see https://docs.github.com/en/repositories/creating-and-managing-repositories/cloning-a-repository

- Add the path to the cloned PASTa repository to MATLAB at the start of each analysis session.

    - For instructions to add folders to the search path, see https://www.mathworks.com/help/matlab/ref/addpath.html


_Note: The file_ main_ExampleAnalysis_Transients.m _in the Example Analyses folder serves as a template for the main script to run functions described in the protocol. An example of adding the cloned PASTa repository folder to path is included at the beginning of the script._

For an overview of how to use GitHub and best practices: https://www.freecodecamp.org/news/introduction-to-git-and-github/

# Get Started with an Example Analysis
To help users get started with PASTa, the repository includes a full example analysis. The example includes all stages of the pipeline from data preparation through signal processing and transient event detection and analysis. For full details, see the [Example Analyses](https://rdonka.github.io/PASTaUserGuide/exampleanalyses/) page.

# Learning Resources
For users unfamiliar with MATLAB, we have compiled some helpful introductory resources on the [Learning Resources](https://rdonka.github.io/PASTaUserGuide/learningresources/) page.

# Contact Us!
Please feel free to reach out with questions, feedback, and feature requests. PASTa is actively maintained, and will continue to integrate new features into future versions. Version releases will be continually added to GitHub, and details are also available on the [Version Release Notes](https://rdonka.github.io/PASTaUserGuide/about/releasenotes/) page.

To facilitate community discussion, we are actively maintaining the [Discussions Forum](https://github.com/rdonka/PASTa/discussions) via GitHub, which includes channels for announcements, feature requests, troubleshooting, and general discussions. Our individual contact information is also available [here](https://rdonka.github.io/PASTaUserGuide/contactus/).


