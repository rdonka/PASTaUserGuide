# Function Documentation Overview
This page contains additional documentation for each function within PASTa, as well as examples of inputs. Detailed documentation is also available within each MATLAB function and can be viewed via the MATLAB help window or by opening the function file.

# Data Preparation Functions
This set of functions is used to prepare raw photometry data, match it with experimental metadata, and load data into a structure in MATLAB. Functions are provided to handle data collected via TDT equipment and software Synapse, or a generic file structure with data streams saved to CSV files.

## createExperimentKey
Combine subject key and file key into a single data structure, appending the provided rootdirectory to create full paths to stored data locations.

**INPUTS:**

* __ROOTDIRECTORY:__ String. The top-level directory path unique to your system. For example: 'C:\Users\rmdon\'. Must end with a slash/backslash.
* __SUBJECTKEYNAME:__ String. The name (or full path) of the subject key CSV file (e.g. 'mySubjectKey.csv'). Must contain at least the variable *'SubjectID'*. To omit a subject key and only load in the file key, set subjectkeyname = ''. If empty (''), the subject key is skipped and only FILEKEYNAME is loaded. See the user guide [Data Preparation](https://rdonka.github.io/PASTaUserGuide/userguide/datapreparation/) for more details.
* __FILEKEYNAME:__ String. The name (or full path) of the file key CSV file (e.g. 'myFileKey.csv'). Must contain *'SubjectID'*, *'BlockFolder'*, *'RawFolderPath'*, and *'ExtractedFolderPath'* columns at minimum. Paths in *'RawFolderPath'* and 'ExtractedFolderPath'should each end with a slash/backslash.

**OUTPUTS:**

* __EXPERIMENTKEY:__ A data structure of the combined file and subject key data. The *'rootdirectory'* is prepended to each row's *'RawFolderPath'* and *'ExtractedFolderPath'*, while the *'BlockFolder'* name is appended to both. 


**EXAMPLE:**
```
rootdirectory = 'C:\Users\rmdon\';
subjKey = 'subjectKey.csv';
fileKey = 'fileKey.csv';

 experimentkey = createExperimentKey(rootdirectory, subjKey, fileKey); % Load keys into a data structure called experimentkey
```

**NOTES:**

* FILEKEY must contain at a minimum the fields _SubjectID_, _BlockFolder_, _RawFolderPath_, and _ExtractedFolderPath_. 
* SUBJECTKEY must contain at a minimum the field _SubjectID_
* Folder paths must end with a slash.
* The subject and file keys are joined based on *SubjectID*. Subject key must contain every subjects in the file key. If there is a mismatch, you will receive an error message that the right table does not contain all the key variables that are in the left table. The error message will display the unique subject IDs present in each key so you can determine where the mismatch occurred.
* Fields in subject and file key must be named uniquely. The only field that should be named the same in both keys is Subject.

## extractTDTdata
This function is used to extract TDT data from saved blocks recorded via the software _Synapse_. For each block, extractTDTdata calls the function *"TDTbin2mat"* (TDT, 2025) and inputs the *RawFolderPath* to extract fiber photometry data recorded with *Synapse*. Extracted blocks are parsed it into a single data structure containing all fields, streams, and event epocs. Extracted signal streams are trimmed to remove the first 5 seconds by default (trimming can be adjusted if desired). 

The function will identify the signal channel by matching the names in the input SIGSTREAMNAMES and the control channel by matching the names in the input BAQSTREAMNAMES. The name inputs can include a list of stream names if channel naming conventions vary by rig. Each block is saved as a separate data structure in a '.mat' file at the location specified by the inputs in *extractedfolderpaths*. 

For more details on preparing TDT data blocks, see the user guide section on [Data Preparation: Data Organization](https://rdonka.github.io/PASTaUserGuide/userguide/datapreparation/#data-organization).

**INPUTS:**

* __RAWFOLDERPATHS:__ String array of raw TDT block folder paths. The string array should contain one column with each full path in a separate row. If using the *createExperimentKey* function, this can be created from the experiment key data structure. __For example:__
```
rawfolderpaths = string({experimentkey.RawFolderPath})';
```
* __EXTRACTEDFOLDERPATHS:__ String array of corresponding output paths where extracted data is saved. If using the createExperimentKey function, this can be created from the experiment key. __For example:__
``` 
extractedfolderpaths = string({experimentkey.ExtractedFolderPath})';
```
* __SIGSTREAMNAMES:__ Cell array of possible stream names for the "signal" channel (e.g. {'x65A','465A'}). __NOTE:__ Only one stream per file can be treated as signal. If different files have different stream names, include all stream names in the cell array.
* __BAQSTREAMNAMES:__ Cell array of possible stream names for the "background" channel (e.g. {'x05A','405A'}). __NOTE:__ Only one stream per file can be treated as background. A cell array containing strings with the names of the streams to be treated as background. If different files have different background stream names, include all stream names in the cell array.

**OPTIONAL INPUTS:**

* __'trim':__ Numeric; Number of seconds to trim from the start and end of each recording. Default: 5
* __'skipexisting':__  Numeric (0 or 1; Default: 1); If 1, skip extracting any session for which an output file already exists. If 0, re-extract and overwrite. This allows the user to toggle whether or not to extract every block, or only blocks that have not previously been extracted. If not specified, defaults to skip previously extracted blocks (1).

**OUTPUTS:**

Saved .mat data structures for each block in the location specified by *extractedfolderpaths*.

**EXAMPLE - DEFAULT:**
```
sigstreamnames = {'x65A', '465A'}; % All names of signal streams across files
baqstreamnames = {'x05A', '405A'}; % All names of background streams across files
rawfolderpaths = string({experimentkey.RawFolderPath})'; % Create string array of raw folder paths
extractedfolderpaths = string({experimentkey.ExtractedFolderPath})'; % Create string array of extracted folder paths

% Extract and save data structures for each file
extractTDTdata(rawfolderpaths,extractedfolderpaths,sigstreamnames,baqstreamnames); 
```

**EXAMPLE - MANUALLY SPECIFIED TRIM AND SKIPEXISTING:**
```
trim = 3; % Set trim to 3 seconds
skipexisting = 0; % Extract all blocks
sigstreamnames = {'x65A', '465A'}; % All names of signal streams across files
baqstreamnames = {'x05A', '405A'}; % All names of background streams across files
rawfolderpaths = string({experimentkey.RawFolderPath})'; % Create string array of raw folder paths'
extractedfolderpaths = string({experimentkey.ExtractedFolderPath})'; % Create string array of extracted folder paths'

% Extract and save data structures for each file
extractTDTdata(rawfolderpaths,extractedfolderpaths,sigstreamnames,baqstreamnames,'trim',trim,'skipexisting',skipexisting); 
```

## extractCSVdata
This function is used to extract data collected from non-TDT systems (e.g., Neurophotometrics, Doric). Data must first be prepared in the generic CSV format, with each session saved as a folder containing CSV files of the signal, background, session recording parameters, and event epochs (optional). Note that session recording parameters must at least include the sampling rate as the variable *fs*. For more details on preparing CSV data files, see the user guide section on [Data Preparation: Data Organization](https://rdonka.github.io/PASTaUserGuide/userguide/datapreparation/#data-organization).

For each block, *extractCSVdata* extracts the signal and background streams, recording parameters, and any specific event epochs into a single data structure containing all recording parameters, streams, and included event epochs. Extracted signal and background streams are trimmed to remove the first 5 seconds by default (trimming can be adjusted if desired). 

Each block is saved as a separate data structure in a '.mat' file at the location specified by the inputs in *extractedfolderpaths*. 

**INPUTS:**

* __RAWFOLDERPATHS:__ String array of raw TDT block folder paths. The string array should contain one column with each full path in a separate row. If using the *createExperimentKey* function, this can be created from the experiment key data structure. __For example:__
```
rawfolderpaths = string({experimentkey.RawFolderPath})';
```
* __EXTRACTEDFOLDERPATHS:__ String array of corresponding output paths where extracted data is saved. If using the createExperimentKey function, this can be created from the experiment key. __For example:__
``` 
extractedfolderpaths = string({experimentkey.ExtractedFolderPath})';
```
* __SIGSTREAMNAME:__ String; Name of csv files containing the "signal" channel (e.g., *'sig'*). __NOTE:__ Only one stream per file can be treated as signal.
* __BAQSTREAMNAME:__ String; Name of csv files containing the "background" channel (e.g. *'baq'*). __NOTE:__ Only one stream per file can be treated as background.

**OPTIONAL INPUTS:**

* __'loadepocs':__ Logical; Set to 1 to load event epoch files in the *'Raw Data'* session folders.
* __'epocsnames':__ Cell array of file names of csv files containing event epochs to be loaded into the data structure. Note: Each type of epoch should be saved to a separate csv file. Pass all event epoch files names to *epocsnames*.
* __'trim':__ Numeric; Number of seconds to trim from the start and end of each recording. Default: 5
* __'skipexisting':__  Numeric (0 or 1; Default: 1); If 1, skip extracting any session for which an output file already exists. If 0, re-extract and overwrite. This allows the user to toggle whether or not to extract every block, or only blocks that have not previously been extracted. If not specified, defaults to skip previously extracted blocks (1).

**OUTPUTS:**

Saved .mat data structures for each block in the location specified by *extractedfolderpaths*.

**EXAMPLE - DEFAULT:**
```
sigstreamname = 'sig'; % All names of signal streams across files
baqstreamname = 'baq''; % All names of background streams across files
rawfolderpaths = string({experimentkey.RawFolderPath})'; % Create string array of raw folder paths
extractedfolderpaths = string({experimentkey.ExtractedFolderPath})'; % Create string array of extracted folder paths

% Extract and save data structures for each file
extractCSVdata(rawfolderpaths,extractedfolderpaths,sigstreamname,baqstreamname); 
```

**EXAMPLE - WITH EPOCHS:**
```
sigstreamname = 'sig'; % All names of signal streams across files
baqstreamname = 'baq''; % All names of background streams across files
epocsnames = {'strt', 'injt'}; % Prepare names of csv files containing event epochs
rawfolderpaths = string({experimentkey.RawFolderPath})'; % Create string array of raw folder paths
extractedfolderpaths = string({experimentkey.ExtractedFolderPath})'; % Create string array of extracted folder paths

% Extract and save data structures for each file
extractCSVdata(rawfolderpaths,extractedfolderpaths,sigstreamname,baqstreamname,'loadepocs',1,'epocsnames',epocsnames); 
```

**EXAMPLE - MANUALLY SPECIFIED TRIM AND SKIPEXISTING:**
```
trim = 3; % Set trim to 3 seconds
skipexisting = 0; % Extract all blocks
sigstreamname = 'sig'; % All names of signal streams across files
baqstreamname = 'baq''; % All names of background streams across files
rawfolderpaths = string({experimentkey.RawFolderPath})'; % Create string array of raw folder paths'
extractedfolderpaths = string({experimentkey.ExtractedFolderPath})'; % Create string array of extracted folder paths'

% Extract and save data structures for each file
extractCSVdata(rawfolderpaths,extractedfolderpaths,sigstreamname,baqstreamname,'trim',trim,'skipexisting',skipexisting); 
```

## loadKeydata
For use with previously extracted data. Compatible with data structures extracted by either the *extractTDTdata* or *extractCSVdata* functions. *loadKeydata* loads previously extracted .mat data blocks with individaul session data into a main data structure for further analysis. Each block is one row. The input experiment key must include at a minimum the extracted folder path location of the block. Any additional information about the subject and session in the experiment key will be matched to the extracted data.

**INPUTS:**

* __EXPERIMENTKEY:__ A prepared data structure with at minimum the ExtractedFolderPath containing the full path to the individual session data structures to be loaded. The EXPERIMENTKEY can be prepared with the LOADKEYS function.

**OUTPUTS:**

* __DATA:__ the input data structure with each individual extracted block added by row.

**EXAMPLE:**
```
[rawdata] = loadKeydata(experimentkey); % Load data based on the experiment key into the structure 'rawdata'
```

## cropFPdata
Used to crop data streams to remove portions that are not to be included in analysis (such as the the first two minutes of the session, or the first samples before a hardware control program is initiated). Crops all specified data streams from the index in *cropstart* to the index in *cropend*, and adjusts specified event epochs by the amount cropped by *cropstart*. Users must pre-prepare the crop start and end indexes to specify as inputs for the function.

**INPUTS:**

* __DATA:__ Data structure array containing at least the specified input fields.
* __CROPSTARTFIELDNAME:__ String; Field name for cropping start index. __For example:__ 'sessionstart'.
* __CROPENDFIELDNAME:__ String; Field name for cropping end index. __For example:__ 'sessionend'.
* __STREAMFIELDNAMES:__ A cell array containing the names (strings) of all the data streams to be cropped. __For example:__ {'sig', 'baq'}

**OPTIONAL INPUTS:**

* __'epocsfieldnames':__ A cell array containing the field names of all the event epochs to be adjusted for cropping (event epoch - (start loc - 1)).

**OUTPUTS:**

* __DATA:__ The data structure with the specified data streams containing the cropped data, and event epochs adjusted to maintain alignment with the data streams.

**EXAMPLE: WITHOUT EVENT EPOCS**
```
cropstartfieldname = 'sessionstart'; % Name of field with session start index
cropendfieldname = 'sessionend'; % Name of field with session end index
streamfieldnames = {'sig', 'baq','time'}; % Names of all streams to crop

% Output cropped data into new structure called data
[data] = cropFPdata(rawdata,cropstartfieldname,cropendfieldname,streamfieldnames); 
```

**EXAMPLE: WITH EVENT EPOCS**
```
cropstartfieldname = 'sessionstart'; % Name of field with session start index
cropendfieldname = 'sessionend'; % Name of field with session end index
streamfieldnames = {'sig', 'baq','time'}; % Names of all streams to crop
epocsfieldnames = {'injt','sess'}; % Field names of event epocs to adjust to maintain relative position

% Output cropped data into new structure called data
[data] = cropFPdata(rawdata,cropstartfieldname,cropendfieldname,streamfieldnames,'epocsfieldnames',epocsfieldnames); 
```


# Signal Processing Functions
This set of functions is used to process raw fiber photometry data prior to further analyses. Functions are available to perform background subtraction, filter, and normalize the data streams. While default options have been chosen to provide users with a conservative starting point, multiple options for signal processing and normalization are available to allow for customization to adapt to experimental needs.

For a full discussion of available signal processing methods, see the user guide section [Signal Processing](https://rdonka.github.io/PASTaUserGuide/userguide/signalprocessing/).

## subtractFPdata
Subtracts background fluorescence stream (eg, 405nm 'baq') from the signal stream (eg, 465nm 'sig'), converts the subtracted signal to ΔF/f, and applies a filter to denoise the output. Users must input at a minimum the data structure with the raw data, the names of the fields containing the signal and background streams, and the name of the field containing the sampling rate of the collected data.

**INPUTS:**

* __DATA:__ Data structure array containing at least the specified input fields (signal and background streams, sampling rate)
* __SIGFIELDNAME:__ String; name of the field in DATA containing the signal data stream (e.g., *'sig'*).
* __BAQFIELDNAME:__ String; name of the field in DATA containing the background data stream (e.g., *'baq'*).
* __FSFIELDNAME:__ String; name of the field in DATA containing the sampling rate of the fiber photometry data streams in Hz (e.g., *'fs'*). __Note:__ The sampling rate must be specified in Hz for filtering to be properly applied.

**OPTIONAL INPUTS:**

* __'baqscalingtype':__ A string to specify the type of background scaling to apply. Options are *'frequency'*, *'sigmean'*, *'OLS'*, *'detrendedOLS'*, *'smoothedOLS'*, *'biexpOLS'*, *'biexpQuartFit'*, or *'IRLS'*. __Default:__ *'frequency'*.
    * _'frequency':_ __Default__; Scales the background to the signal channel based on ratio of the power in the specified frequency bands in the FFT (frequency domain) of the channels.
    * _'sigmean':_ Scales the background to the signal channel based on the ratio of the mean of the signal to the mean of the background (time domain).
    * _'OLS':_ Uses ordinary least-squares regression to generate scaled background.
    * _'detrendedOLS':_ Removes the linear trend from signal and background streams prior to using ordinary least-squares regression to generate scaled background.
    * _'smoothedOLS':_ Applies lowess smoothing to the signal and background streams prior to using ordinary least-squares regression to generate scaled background.
    * _'biexpOLS':_ Fits and removes a bi-exponential decay function from the raw background and signal streams, then scales the corrected background to the corrected signal with ordinary least-squares regression.
    * _'biQuartFit':_ Fits and removes a bi-exponential decay function from the raw background and signal streams, then scales the corrected background to the corrected signal based on the interquartile range.
    * _'IRLS':_ Uses iteratively reweighted least squares regression to generate scaled background.
* __'baqscalingfreqmin':__ Only used with *'frequency'* scaling. Numeric frequency (Hz) minimum threshold for scaling the background to signal channel. Frequencies above this value will be included in the scaling factor determination. __Default:__ 10 Hz.
* __'baqscalingfreqmax':__ Only used with *'frequency'* scaling. Numeric frequency (Hz) maximum threshold for scaling the background to signal channel. Frequencies below this value will be included in the scaling factor determination. __Default:__ 100 Hz.
* __'baqscalingperc':__ Only used with *'frequency'* and *'sigmean'* scaling. Adjusts the background scaling factor to be a percent of the derived scaling factor value. __Default:__ 1 (100%).
* __'subtractionoutput':__ Output type for the subtracted data. __Default:__ 'dff'
    * _'dff':_ Outputs subtracted signal as ΔF/F. ((signal - scaled background) / scaled background)
    * _'df':_ Outputs subtracted signal as ΔF. (signal - scaled background)
* __'artifactremoval'__: Logical (0 or 1); set to 1 to detect and remove artifacts from data streams. Default: 0 (false). _Note:_ see *removeStreamArtifacts* function documentation for more detail. 
* __'filtertype':__ A string to specify the type of filter to apply after subtraction. __Default:__ 'bandpass'.
    * _'nofilter':_ No filter will be applied.
    * _'bandpass':_ A bandpass Butterworth filter will be applied.
    * _'highpass':_ Only a high pass Butterworth filter will be applied.
    * _'lowpass':_ Only a low pass Butterworth filter will be applied.
* __'padding':__ Defaults to 1, which applies padding prior to filtering. Padding takes the first 10% of the stream, flips it, and appends it to the data before applying the filter to prevent edge effects. Appended data is trimmed after filtering. Set to 0 to turn off padding of data streams. __Default:__ 1.
* __'paddingperc':__ Percent of data length to use to determine the number of samples to be appended to the beginning and end of data in padding. Set to minimum of 0.1 (10%). __Default: 0.1__ (10%).
* __'filterorder':__  The order to be used for the chosen Butterworth filter. __Default:__ 3.
* __'highpasscutoff':__  The cutoff frequency (Hz) to be used for the high pass Butterworth filter. __Default:__ 2.2860.
* __'lowpasscutoff':__ The cutoff frequency (Hz) to be used for the low pass butterworth filter. __Default:__ 0.0051.
    * NOTE: *'bandpass'* applies both the high and low pass cutoffs to design the filter.


**OUTPUTS:**

* __DATA:__ The original data structure with added fields with the scaled background ('baqscaled'), subtracted signal ('sigsub'), and subtracted and filtered signal ('sigfilt'). All inputs and defaults will be added to the data structure under the field 'inputs'.
    * __NOTE:__ If using BAQSCALINGMETHOD _'detrendedOLS'_, additional fields containing the detrended signal and background ('sigdetrend' and 'baqdetrend') will be added to the data frame. If using BAQSCALINGMETHOD _'smoothedOLS'_, additional fields containing the smoothed signal and background ('sigsmoothed' and 'baqsmoothed') will be added to the data frame. If using BAQSCALINGMETHOD _'biexpOLS'_ or _'biexpQuartFit'_, additional fields containing the bi-exponential decay corrected signal and background ('sigbiexpcorrected' and 'baqbiexpcorrected') will be added to the data frame.  


**EXAMPLE - DEFAULT:**
```
sigfieldname = 'sig';
baqfieldname = 'baq';
fsfieldname = 'fs';

data = subtractFPdata(data,sigfieldname,baqfieldname,fsfieldname);
```

**EXAMPLE - Frequency Scaling with 20Hz Minimum Threshold and Highpass Filter Only:**
```
sigfieldname = 'sig';
baqfieldname = 'baq';
fsfieldname = 'fs';

data = subtractFPdata(data,sigfieldname,baqfieldname,fsfieldname,'baqscalingfreqmin',20,'filtertype,'highpass');
```

**EXAMPLE - OLS Scaling:**
```
sigfieldname = 'sig';
baqfieldname = 'baq';
fsfieldname = 'fs';

data = subtractFPdata(data,sigfieldname,baqfieldname,fsfieldname,'baqscalingtype','OLS');
```

**EXAMPLE - IRLS Scaling:**
```
sigfieldname = 'sig';
baqfieldname = 'baq';
fsfieldname = 'fs';

data = subtractFPdata(data,sigfieldname,baqfieldname,fsfieldname,'baqscalingtype','IRLS');
```

## removeStreamArtifacts
Detects and removes artifacts from fiber photometry data streams. Called by the *subtractFPdata* function prior to filtering if the optional input *removeartifacts* is set to 1.

**INPUTS:**

* __DATASTREAM:__ Numeric array; The signal data for artifact removal.
* __STREAMFIELDNAME:__ String; Name of the stream being processed.
* __FS:__ Numeric; Sampling rate of the data stream being processed (Hz).

**OPTIONAL INPUTS:**

* __'outlierthresholdk':__ ANumeric, Integer; Multiplier for outlier threshold detection. __Default:__ 3
* __'artifactremovalwindow':__  Numeric; Seconds before and after an artifact to replace with NaNs. __Default:__ 0.3
* __'artifactampthresh_max':__ Numeric; Standard deviation threshold for high artifacts. __Default:__ 8
* __'artifactampthresh_min':__ Numeric; Standard deviation threshold for low artifacts. __Default:__ 8
* __'bucketsizeSecs':__ Numeric; Bucket time window length (seconds) for amplitude and mean calculations.

**OUTPUTS:**

* __ARTIFACTREMOVALDATA:__ Data structure containing:
    * __STREAMFIELDNAME_AR:__ Data stream with artifacts replaced by NaNs
    * __STREAMFIELDNAME_AM:__ Data stream with artifacts replaced by mean values
    * __NUMARTIFACTS:__ Number of detected artifacts
    * __ARTIFACTS:__ Table of artifact details (locations, values, start/end indices)


**EXAMPLE: Default Parameters**
```
datastream = [data(1).sigsub]; % Prepare numeric vector of a stream for one recording session
streamfieldname = 'sigsub'; % Field name of the prepared *datastream* input for artifact removal
fs = data(1).fs; % Prepare sampling rate of the data stream in Hz

% Detect and Remove Artifacts
[artifactremovaldata] = removeStreamArtifacts(datastream,streamfieldname,fs);
```

**EXAMPLE: Increased Artifact Threshold**
```
datastream = [data(1).sigsub]; % Prepare numeric vector of a stream for one recording session
streamfieldname = 'sigsub'; % Field name of the prepared *datastream* input for artifact removal
fs = data(1).fs; % Prepare sampling rate of the data stream in Hz

% Detect and Remove Artifacts
[artifactremovaldata] = removeStreamArtifacts(datastream,streamfieldname,fs,'artifactampthresh_max',10,'artifactampthresh_min',10);
```

## prepareStreamFFT
Helper function to compute the one-sided Fast Fourier Transform (FFT) of a data stream for analysis and plotting. This function is called by the *subtractFPdata*, *plotFFTmag*, and *plotFFTpower* functions.

**INPUTS:**

* __STREAMDATA:__ Numeric vector; the input data stream to be transformed.
* __FS:__ Positive scalar; the sampling rate of the data stream in Hz.


**OUTPUTS:**

* __STREAMFFT:__ Numeric vector; the magnitudes of the one-sided FFT of the input data stream.
* __STREAMF:__ Numeric vector; the corresponding frequency values in Hz for the FFT magnitudes.

**EXAMPLE:**
```
streamdata = [data(1).sig]; % Prepare numeric vector of the signal stream for one recording session
fs = data(1).fs; % Prepare sampling rate of the data stream in Hz

% Compute the FFT
[streamFFT, streamF] = preparestreamFFT(streamdata, fs);
```

## normSession
Normalizes a specified data stream to Z-score based on the entire session mean and standard deviation.

**INPUTS:**

* __DATA:__ Data structure array; each element (row) represents a single recording session and must contain the field specified by STREAMFIELDNAME.
* __STREAMFIELDNAME:__ String; the name of the field within DATA to be normalized. __For example:__ 'sigfilt'


**OUTPUTS:**

* __DATA:__ The original data structure with the added field 'data.WHICHSTREAMz_normsession' containing the whole session normalized signal.

**EXAMPLE:**
```
[data] = normSession(data,'sigfilt'); % Outputs whole session z score
```

## normBaseline
Normalizes a specified data stream to Z-score based on the specified session baseline period.

**INPUTS:**

* __DATA:__ Data structure array; each element (row) represents a single recording session and must contain the field specified by STREAMFIELDNAME.
* __STREAMFIELDNAME:__ String; the name of the field within DATA to be normalized. __For example:__ 'sigfilt'
* __BLSTARTFIELDNAME:__ String; The name of the field containing the index of the start of the baseline period.
* __BLENDFIELDNAME:__ String; The name of the field containing the index of the end of the baseline period.


**OUTPUTS:**

* __DATA:__ The original data structure with the added field 'data.WHICHSTREAMz_normbaseline' containing the baseline normalized signal.

**EXAMPLE:**
```
% Prepare baseline start and end indices based on event epoch
for eachfile = 1:length(data)
    data(eachfile).BLstart = 1;
    data(eachfile).BLend =  data(eachfile).injt(1);
end

% Normalize filtered signal to session baseline mean and SD
[data] = normBaseline(data,'sigfilt','BLstart','BLend');
```

## normCustom
Normalizes the whole session data stream based on a custom period input as a separate stream field.

**INPUTS:**

* __DATA:__ Data structure array; each element (row) represents a single recording session and must contain the field specified by STREAMFIELDNAME.
* __FULLSTREAMFIELDNAME:__ String; the name of the field within DATA to be normalized. __For example:__ 'sigfilt'
* _CUSTOMSTREAMFIELDNAME:__ String; The name of the field containing the cut data stream to use as the reference for normalization.

**OUTPUTS:**

* __DATA:__ The original data structure with the added field 'data.WHICHSTREAMz_normcustom' containing the baseline normalized signal.

**EXAMPLE:**
```
% Prepare custom data stream for normalization using the pre-trial and post-trial periods of the session
for eachfile = 1:length(data)
    data(eachfile).customnorm = [data(eachfile).sigfilt(1:data(eachfile).trial(1)), data(eachfile).sigfilt(data(eachfile).trial(30):length(data(eachfile).sigfilt))];
end

% Normalize filtered signal to custom session period mean and SD
[data] = normCustom(data,'sigfilt','customnorm');
```


## preparestreamFFT
Helper function to take the FFT of a data stream and output the magnitudes and corresponding frequencies for analysis, plotting, and diagnostics. This function is called by the plot function *plotFFTs* but can also be used to prepare FFTs for each stream of data for all sessions with use in a for loop.

**INPUTS:**

* __STREAMDATA:__ An array containing the values from a single collected data stream.
* __FS:__ The sampling rate of the data stream.


**OUTPUTS:**

* __STREAMFFT:__ An array containing the prepared FFT magnitudes of the input stream.
* __STREAMF:__ An array containing the corresponding frequencies to the prepared FFT magnitudes in STREAMFFT.

**EXAMPLE OF SINGLE SESSION SIGNAL:**
```
[streamFFT,streamF] = preparestreamFFT(data(1).sig,data(eachfile).fs);
```

**EXAMPLE OF ALL SESSIONS AND STREAMS:**
```
%% Prepare FFTs of all streams
streams = {'sig', 'baq', 'baq_scaled','sigsub', 'sigfilt'};

% Frequency scaled background
for eachfile = 1:length(data)
    for eachstream = 1:length(streams)
        streamname = char(streams(eachstream));
        [streamFFT,streamF] = preparestreamFFT(data(eachfile).(streamname),data(eachfile).fs);
        data(eachfile).(append(streamname,'FFT')) = streamFFT;
        data(eachfile).(append(streamname,'F')) = streamF;
    end
end
```

# Transient Analysis Functions
## findTransients
Main function to detect and quantify transient events in whole session data streams. Users must input the raw data structure containing the data stream, a field with threshold values for transient inclusion, and the name of the field with the sampling rate of the stream. *findTransients* will output a new data structure (*transientdata*) with any user specified fields from the original data structure, and the field *'transientquantification'* containing quantification variables for each detected transient event. 

The *findTransients* function will set default values for transient detection and quantification parameters not specific as optional inputs. Optional inputs can be specified to modify transient detection parameters. See the user guide section [Transient Analysis](https://rdonka.github.io/PASTaUserGuide/userguide/transientanalysis/) for more details.

**INPUTS:**

* __DATA:__ Data structure array; each element (row) represents a single recording session and must contain the fields specified by ADDVARIABLESFIELDNAMES, STREAMFIELDNAME, THRESHOLDFIELDNAME, and FSFIELDNAME.
* __ADDVARIABLESFIELDNAMES:__ Cell array; names of the fields in DATA to add to the new data structure. This should include SubjectID and experimentally relevant metadata. __For example:__ {'SubjectID', 'BlockFolder', 'Dose'}
* __STREAMFIELDNAME:__ String; name of the field in DATA containing the data stream to be analyzed for transient events (e.g., 'sigfiltz_normsession')
* __THRESHOLDFIELDNAME:__ String; name of the field in DATA containing the numeric threshold values for transient detection (e.g., 'SDthreshold'). Thresholds should be precomputed and typically set to 2.6 standard deviations
* __FSFIELDNAME:__ String; name of the field in DATA containing the sampling rate (fs) of the data stream.


**OPTIONAL INPUTS:**

* __'bltype':__ String; Method for pre-transient peak baseline detection. Options are *'blmean'* (__default__), *'blmin'*, and *'localmin'*.
    * _'blmean':_ Pre-transient baselines are set to the mean of the pre-transient window.
    * _'blmin':_ Pre-transient baselines are set to the minimum value within the pre-transient window.
    * _'localmin':_ Pre-transient baselines are set to the local minimum directly preceding the transient within the baseline window.
* __'blstartms'__ Numeric; start time (ms) of the pre-transient baseline window. __Default:__ 1000 ms
* __'blendms'__ Numeric; end time (ms) of the pre-transient baseline window. __Default:__ 200 ms
* __'posttransientms'__ Numeric; duration (ms) after the transient peak for analysis. __Default:__ 2000 ms
* __'quantificationheight'__ Numeric; height (as a fraction of peak amplitude) at which to characterize rise time, fall time, peak width, and area under the curve (AUC). Must be between 0 and 1. __Default:__ 0.5
* __'compoundtransientwindowms':__ Numeric; window (ms) to search before and after each event for compound transients. __Default:__ 2000 ms.
* __'outputtransientdata':__ Logical; if true (1), outputs cut data streams for each transient event. If false (0), skips this output. __Default:__ true (1)
* __'outputpremaxS':__ Numeric; Number of seconds pre transient peak to include in transient data output streams. __Default:__ 3
* __'outputpostmaxS':__ Numeric; Number of seconds post transient peak to include in transient data output streams. __Default:__ 5


**OUTPUTS:**

* __TRANSIENTDATA:__ Structure array; each element corresponds to a session and includes all fields specified in ADDVARIABLESFIELDNAMES as well as the following:
    * __params.findTransients:__ Structure of input parameters used for transient detection.
    * __transientquantification:__ Table of quantified variables for each transient, including amplitude, rise time, fall time, width, and AUC.
    * If OUTPUTTRANSIENTDATA is set to 1:
        * __transientstreamlocs:__ Table of pre-transient baseline, transient peak, rise, and fall locations for each transient.
        * __transientstreamdata:__ Table of cut data streams from baseline start to the end of the post-transient period for each transient event.


**NOTES:**

* Threshold values should be calculated before using the _findTransients_ functions. We reccomend setting the threshold to 2.6 SDs. Typically thresholds are set between 2-3 SDs. If the input data stream is Z-scored, thresholds can be set to the actual SD threshold number. If the input data stream is not Z-scored, find the corresponding value to 2-3 SDs for each subject.
* For transient data outputs, each transient is in a separate row. 
* If OUTPUTTRANSIENTDATA is set to anything other than 1, the TRANSIENTSTREAMLOCS and TRANSIENTSTREAMDATA tables will be skipped and not included in the output.


**EXAMPLE:**
```
% Prepare thresholds for Z-scored data stream
for eachfile = 1:length(data)
    data(eachfile).thresholdSD = 2.6;
end

% Find session transients based on pre-peak baseline window minimum
[data] = findTransients(data,'sigfiltz_normsession_trimmed','thresholdSD','fs');

% Find session transients based on pre-peak baseline window minimum
[data] = findTransients(data,'sigfiltz_normsession_trimmed','threshold3SD','fs','bltype','blmean');

% Find session transients based on pre-peak local minimum (last minumum before the peak in the baseline window)
[data] = findTransients(data,'sigfiltz_normsession_trimmed','threshold3SD','fs','bltype','localmin');
```


## binTransients
Used to assign individual transient events to time bins within the session for analysis of changes in transients over time. By default, sessions will be divided into 5 minute bins. Number of bins per session will be calculated by the function. Users have the option to override the defaults to change the bin length and/or manually set the number of bins per session.

**INPUTS:**

* __TRANSIENTDATA:__ Structure array output from *findTransients*. Must contain *'params.findTransients'* and *'transientquantification'* fields.
* __WHICHSTREAM:__ The name (string) of the field containing the stream to be analyzed for transients.
* __WHICHFS:__ The name (string) of the field containing the sampling rate of the raw data collection in Hz.
* __WHICHTRANSIENTS:__ The name (string) of the parent field containing the table of transietns that you want to identify bins for. This is the output of the _findSessionTransients_ function. The name usually follows the convention *sessiontransients_WHICHBLTYPE_WHICHTHRESHOLD*


**OPTIONAL INPUTS:**
* __WHICHTRANSIENTSTABLE:__ The name (string) of the field within WHICHTRANSIENTS that contains the quantification of individual transient events. This input only needs to be specified if not using the format output from the FINDSESSIONTRANSIENTS functions. _Default: 'transientquantification'_
* __WHICHMAXLOCS:__ NThe name (string) of the field containing the transient max locations (indexes) relative to
the whole session. This input only needs to be specified if not using the format output from the FINDSESSIONTRANSIENTS functions. _Default: 'maxloc'_
* __BINLENGTHMINS:__ Numeric; Bin length in number of minutes. _Default: 5_
* __NBINSOVERRIDE:__ Numeric; Manual override to set the number of bins. If set to anything other than 0, users can override the stream-length based calculation of the number of bins per session and set their own number. _Default: 0_


**OUTPUTS:**

* __DATA:__ The original data structure with sessiontransients_localmin_THRESHOLDLABEL added in. For more details on individual variables, see the PASTa user guide page _Transient Detection_. The output contains four nested tables: 
    * _INPUTS:_ Includes all function inputs.
    * _TRANSIENTQUANTIFICATION:_ Includes the quantified variables for each transient, including amplitude, rise time, fall time, width, and AUC. 
    * _TRANSIENTSTREAMLOCS:_ Pre-transient baseline (local minimum), transient peak, rise, and fall locations for each transient to match the cut transient stream data.
    * _TRANSIENTSTREAMDATA:_ Cut data stream from baseline window start to the end of the post-transient period for each transient event.


**NOTES:**

* If called outside the main _findSessionTransients_ function, note that users will have to input all parameters manually and no defaults will be applied.
* As for the main function, threshold values should be calculated before using the findSessionTransients functions. Typically thresholds are set to 2-3 SDs. If the input data stream is Z scored, this can be the actual SD threshold number. If the input data stream is not Z scored, find the corresponding value to 2-3 SDs for each subject.
* For transient data outputs, each transient is in a separate row. 
* If OUTPUTTRANSIENTDATA is set to anything other than 1, the TRANSIENTSTREAMLOCS and TRANSIENTSTREAMDATA tables will be skipped and not included in the output.


**EXAMPLE:**
```
% Prepare thresholds - since Z scored streams will be analyzed, input threshold as the desired SD.
for eachfile = 1:length(data)
    data(eachfile).threshold3SD = 3;
end

% Find session transients by directly called in the findSessionTransients_localmin function
[data] = findSessionTransients_localmin(data,'sigfiltz_normsession_trimmed','threshold3SD','fs',800,100,2000,0.5,1);
```

## exportSessionTransients
Used to create a table of all transient events across all sessions included in the file key and data structure, and exports the table of transient quantification to a csv file. This function depends on the output from _findSessionTransients_. The creation of a csv file is helpful for users who prefer to run statistical analyses or make plots in other programs than MATLAB, such as R Studio.

**INPUTS:**

* __DATA:__ This is a structure that contains at least the output from _findSessionTransients_.
* __WHICHTRANSIENTS:__ The name (string) of the parent field containing the table of transietns that you want to identify bins for. This is the output of the _findSessionTransients_ function. The name usually follows the convention *sessiontransients_WHICHBLTYPE_WHICHTHRESHOLD*
* __EXPORTFILEPATH:__ A string with the path to the folder location where the created table should be saved to. NOTE: The path must end in a forward slash.
* __ADDVARIABLES:__ A cell array containing any additional variables from the data structure to be added to the transients table. Variables will be added to every row of the output structure. Cell array inputs must be the names of fields in the data structure. At a minimum, this should contain the subject ID. If multiple sessions per subject are included in the data structure, make sure a session ID variable is also included. For example: {'Subject', 'SessionID', 'Treatment'}


**OPTIONAL INPUTS:**
* __WHICHTRANSIENTSTABLE:__ The name (string) of the field within WHICHTRANSIENTS that contains the quantification of individual transient events. This input only needs to be specified if not using the format output from the FINDSESSIONTRANSIENTS functions. _Default: 'transientquantification'_
* __FILENAME:__ A string with a custom name for the output csv file. If not specified, the file name will be generated as *'WHICHTRANSIENTS_AllSessionExport_DAY-MONTH-YEAR.csv'*


**OUTPUTS:**

* This function outputs a csv file with all transients for all sessions in the data structure. The file will be output at the specified file path. 
* If the function is called into an object, the table ALLTRANSIENTS will also be saved to an object in the MATLAB workspace. 


**NOTES:**

* The file path must include the full path for the folder including the computer address, and must end in a forward slash. If the file path is not specified properly, the function won't be able to automatically save the csv file.
* Make sure to add all relevant experimental variables to the ADDVARIABLES list. This includes at a minimum the Subject ID and a session ID, but it's wise to include other key variables to make future analyses easier. Alternatively, you can keep only the Subject ID and a session ID and match the data to the subject and file keys later, which would include all subject and experimental variables.


**EXAMPLE:**
```
% Export transients with added fields for subject and treatment=
addvariables = {'Subject','TreatNum','InjType'}; % Prepare variables to append to each transient

% To just output the csv file:
exportSessionTransients(data,'sessiontransients_blmean_threshold3SD',analysispath,addvariables);

% To output the csv file and add the created table to a MATLAB data structure:
alltransients = exportSessionTransients(data,'sessiontransients_blmean_threshold3SD',analysispath,addvariables);
```


# Visualization and Plotting Functions
## plotTraces
Main function to plot whole session fiber photometry traces. This function will plot the streams sig, baq, baq_scaled, sigsub, and sigfilt for a single session. Use this function in a loop to plot streams for all sessions in the data structure. This plot function plots the data streams with time in minutes on the x axis. To modify the plot or add additional features like session relevant time stamps, use the plot in a loop. Modified plots must be saved in the external loop. If modifications are not neccesaary, then optional inputs can be specified to directly output and save the plot to a filepath location.


**INPUTS:**

* __DATA:__ This is a structure that contains at least the data stream you want to analyze, a field with the threshold values, and a field with the sampling rate of the data stream.
* __WHICHFILE:__ The file number to be plotted.
* __MAINTITLE:__ The main title (string) to be displayed for the overall plot above the individual tiles. For example, '427 - Treatment: Morphine'


**OPTIONAL INPUTS:**

* __SAVEOUTPUT:__ Set to 1 to automatically save trace plots as png to the plot file path. _Default: 0_
* __PLOTFILEPATH:__ Required if SAVEOUTPUT is set to 1. The specific path to save the plot to. Note that this must be the entire path from computer address to folder ending in the filename for the specific plot. For example: 'C:\Users\rmdon\Box\Injection Transients\Figures\SessionTraces_427_Morphine.png'

**OUTPUTS:**

* __ALLTRACES:__ A plot object containing subplots for each input stream.

**EXAMPLE: Single Session**
```
%% Plot the first session and automatically save the output
maintitle = append(num2str(data(1).Subject),' - Treatment: ',data(1).InjType); % Create title string for current plot
plotfilepath = 'C:\Users\rmdon\Box\Injection Transients\Figures\SessionTraces_427_Morphine.png';  % Create plot file path for the current plot

plotTraces(data,1,maintitle,'saveoutput',1,'plotfilepath',plotfilepath);
```


**EXAMPLE: All Sessions**
```
%% Plot whole session streams for each file
for eachfile = 1:length(data)
    maintitle = append(num2str(data(eachfile).Subject),' - Treatment: ',data(eachfile).InjType); % Create title string for current plot
    alltraces = plotTraces(data,eachfile,maintitle);
    for eachtile = 1:5
        nexttile(eachtile)
        xline(data(eachfile).injt(1),'--','Injection','Color','#C40300','FontSize',8)
        xline(data(eachfile).injt(2),'--','Color','#C40300','FontSize',8)
    end    

    set(gcf, 'Units', 'inches', 'Position', [0, 0, 8, 9]);
    plotfilepath = append(figurepath,'SessionTraces_',num2str(data(eachfile).Subject),'_',data(eachfile).InjType,'.png');
    exportgraphics(gcf,plotfilepath,'Resolution',300)
end
```

## plotFFTs
Main function to plot whole session fiber photometry FFT frequency magnitude plots. This function will plot the streams sig, baq, baq_scaled, sigsub, and sigfilt for a single session. Use this function in a loop to plot streams for all sessions in the data structure. By default, only frequencies up to 20 hz will be plotted but users can specify an optional input to adjust the cutoff lower or higher, or set to the actual max for the stream FFT. To modify the plot or add additional features, use the plot in a loop. Modified plots must be saved in the external loop. If modifications are not neccesaary, then optional inputs can be specified to directly output and save the plot to a filepath location.

**INPUTS:**

* __DATA:__ This is a structure that contains at least the data stream you want to analyze, a field with the threshold values, and a field with the sampling rate of the data stream.
* __WHICHFILE:__ The file number to be plotted.
* __MAINTITLE:__ The main title (string) to be displayed for the overall plot above the individual tiles. For example, '427 - Treatment: Morphine'
* __WHICHFS:__ The name (string) of the field that contains the sampling rate of the data streams.

**OPTIONAL INPUTS:**

* __XMAX:__ The desired frequency cutoff for the x axis (hz). Frequencies above this value will be excluded from the plots. To plot all frequencies, set to actual'. _Default: 20_
* __SAVEOUTPUT:__ Set to 1 to automatically save trace plots as png to the plot file path. _Default: 0_
* __PLOTFILEPATH:__ Required if SAVEOUTPUT is set to 1. The specific path to save the plot to. Note that this must be the entire path from computer address to folder ending in the filename for the specific plot. For example: 'C:\Users\rmdon\Box\Injection Transients\Figures\SessionTraces_427_Morphine.png'

**OUTPUTS:**

* __ALLFFTS:__ A plot object containing subplots for each input stream.

**EXAMPLE: Single Session**
```
%% Plot the first session and automatically save the output
maintitle = append(num2str(data(1).Subject),' - Treatment: ',data(1).InjType); % Create title string for current plot
plotfilepath = 'C:\Users\rmdon\Box\Injection Transients\Figures\SessionTraces_427_Morphine.png';  % Create plot file path for the current plot

plotTraces(data,1,maintitle,'fs','saveoutput',1,'plotfilepath',plotfilepath);
```


**EXAMPLE: All Sessions**
```
%% Plot stream FFTs for each file
for eachfile = 1:length(data)
    maintitle = append(num2str(data(eachfile).Subject),' - Treatment: ',data(eachfile).InjType); % Create title string for current plot
    allffts = plotFFTs(data,eachfile,maintitle);

    set(gcf, 'Units', 'inches', 'Position', [0, 0, 8, 9]);
    plotfilepath = append(figurepath,'SessionFFTs_',num2str(data(eachfile).Subject),'_',data(eachfile).InjType,'.png');
    exportgraphics(gcf,plotfilepath,'Resolution',300)
end
```