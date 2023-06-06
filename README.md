# RF-Pose-Estimation
My Masters Project, under Purdue University,ECE 695:Ideas to Innovation.

Data Collection:
	We started to collect the data in the EE lab building. The main purpose is to set up the parameters correctly and test the stability of the communication between the radar device and the software end. The data that we collected in the lab showed that the multipath propagation of the signals caused much noise to the data we collected. After that, we tried to collect data in the hallway of the EE building but from the data plots we collected, the situation had not improved.
	After knowing the issue caused by the multipath propagation of the signals, we chose to collect our data in an open area. We moved our device to the backyard of one of our team members’ residences so that the tester/driver will be the only object that the radar may detect within 15 meters. The driver was positioned 0.8m ~ 1m away from the radar device and faced directly to the radar. 
And we customized the parameters on mm-Wave Studio to gain maximum radar resolution based on the limitation of the radar device as below:
2 LVDS lanes for receiving reflected signals
Sampling rate - 4400 ksps 
ADC Samples per chirp - 256
Slope - 60.012
Number of chirp per frame - 128
Number of frames -  1800
Periodicity - 100 ms
Format of the received signals - complex
In this way, the noise caused by multipath propagation of signals was decreased significantly. By collecting the data in this environment, we collected 6 actions for each of the 5 individuals from our team and stored them separately labeled the raw data files by different classes for further data processing work.
However, we thought that it was necessary to collect some data under a more realistic or practical environment since ideally our prototype should be installed and applied in a vehicle, a set of data was collected inside of a stopped vehicle as well. We placed the radar device above the dashboard of the vehicle and collected 5 actions from 1 individual of the team.
	We also achieved the automation of setting the parameters and capturing the data by modifying .lua scripts and .py scripts we found online. With the help of the automation modification, we did not need to set up the parameters one by one every time we reboot the mm-Wave Studio IDE, everything can be done just by one click of running the .lua script we modified.

Data Pre-Processing:
	After we have collected all the data we need, we organized the data into folders as figures shown below, following this format:
	Folder name: [action integer label]_[individual Initial]_[action string name]
		File contains:
time-doppler : one time_fre.npy file
Range-doppler & Angle: 1800 .npy files
 	For labeling the actions/bad driving behaviors, we used numbers following the mapping rules:
		0 - Head Left(HL)
		1 - Head Right(HR)
		2 - Leaning Forward(LF)
		3 - Leaning Backward(LB)
		4 - Phone Usage(PU)
		5 - Interference(PI)
	After this, we started to pre-process the raw data we collected . Because our raw ADC data was stored in a frame tensor, it has a chirp axis and ADC sample axis. We applied FFT along the sample axis of our frame tensor to obtain the range-time tensor for further transformation. To calculate the range-doppler plot, we applied FFT along the chirp axis of the range-time tensor. To calculate the angle plot, we measure the phase difference using 2 Rx. Short-time Fourier transforms (STFT) is performed on the signal in the range of focus to get the time-Doppler spectrogram. The range of focus here is obtained by finding the maximum range for every chirp axis. 

Denoising:
	Data collected in enclosed environments(in a lab or inside a car) had a significant amount of noise in them, which required a proper denoising process for accurate predictions by the NN. To create a cleaner plot, each .npy frame was channeled through a Savitzky-Golay filter, to provide a smoothing effect over the time domain. Furthermore, in an effort to combat the multipath propagation in the frequency domain, we decided to compute the FFT plots using a Blackman-Harris window instead of the previous hamming window.

Neural Network:
	For the most important part of our project, we created 2 neural network models. The first one is the TD model , The TD Net takes as an input the augmented TD data from the dataset class I described earlier. The CNN block contains a convolutional layer, batch normalization layer, ReLU activation function, and Maxpool layer. And then each residual block contains two convolutional layers, two batch normalization layers, and two ReLU activation functions.
For the TD neural network model, we created 50 learnable layers with 759,558 learnable parameters and we thought we should have more learnable layers and fewer learning parameters. 
	By using this neural network, we were able to generate the loss plot and evaluation accuracy results.
For the data we collected outdoors, all classification was done with noisy data. The accuracy came out to be lower than we thought because Head Left and Head Right actions are too similar for the network to differentiate since within the process of collecting the data of these two actions, the driver/tester would always need to turn his/her head to left and right to finish one time of Head Left or Head Right action. Same with Leaning Forward and Leaning Backward. 
The model also had difficulty distinguishing between Phone Usage and head movements because turning the head is part of the Phone Usage action.
	For the data we collected in a car, All classification was done with Noisy data as well. However, the Car data was only for one person, for 5 actions. So the accuracy was high because the variety of data was limited, and it was evaluated on only 16 images for each action. 
Next is the RDA model, which takes a combined input of the RD and A data, and then separates them to put them through separate heads. The CNN blocks contain a convolutional layer, batch normalization layer, ReLU activation function, and Maxpool layer. Each residual block contains two convolutional layers, two batch normalization layers, and two ReLU activation functions.
For the RDA Neural Network Model, we created 98 learnable layers with 1,682,974 learnable parameters and here, the idea of “more learnable layers and fewer learning parameters” should be followed as well. 
