# FingerPulseLatency
A Javascript based HTML program to calculate the latency between electrical activity of the heart (R wave from the Polar H10 ECG) to the point of peak blood flow rate (PPG signal from the Polar Verity Sense).

# Instructions
There is only a single HTML file which contains all of the code.

Download and open it within Chrome.

Click the Connect to Polar Verity Sense and select your device. Once the button is green a Bluetooth connection has been established. At that point, if you don't have an H10, you can start SDK mode (first) using the checkbox and then also the PPG and ACC sensors. Data is stored on the local machine and can be downloaded into your filesystem using the button. 

Graphs display 'reduced' data using a Ramer-Douglas-Peucker algorithm to reduce processor load, but full data is also collected for analysis.

Peak detection is dependent on the graph scaling (altered using the + and = keys). If you have an H10 you can also connect to that and display available streams (use the start all streams button to activate everything).

Temporal alignment between the H10 and Verity Sense can be done by holding them together and exposing them to synchronous acceleration. Then align the ACC traces using either the HTML button or the <> keys. Check the alignment, you may need several attempts. For those using the program to correlate with Blood pressure, there are some features for that.

There is essentially no documentation within the code and I am not a programmer. This utility was written for a student doing a final year project. 'Real' programmers may experience mixed emotions on viewing the code for which I apologize.

# Logic involved in decoding frame data
For those struggling to decode the PPG frame data I include below a little bit of information which might help.

The routine completeDeltaFrame is used to 'deconvolve' the frame data. Frame data consists of a header with the starting values for the channels and then values for the bit depth and number of offsets (https://github.com/polarofficial/polar-ble-sdk/blob/master/technical_documentation/Polar_Measurement_Data_Specification.pdf). The bit depth can change markedly if the signal varies greatly (range 8-20 bits in my hands) - i.e. it can be a very large number of bits. Developing a routine that can take a block of byte data and read it out as numbers of a given bit length is the key. Here is my solution (including dealing with the header and frame data):

For the PPG it assumes that the data structure starts with four channels each with data consuming 3 bytes. That allows the offsets between the headers of each packet to be calculated (10+4*3) and the start of the delta frame as well as loading in those intitial sensor values (getInitialSensorValues) that will then have the offsets applied. Then I loop through the data structure extracting the details of each delta frame (DeltaFrameDescription) which are used to calculate the position of the next header and also extract the data from the delta frame (reSlice and addDeltaFrame). The reSlice function or more specifically the ChunkArray function requires some explannation since it repackages the deltaframe data into a form that can be easily read irrespective of the bit depth of the deltaframe data (which is variable).

ChunkArray builds an array of quaternary numbers from a byte array using ChunkByte. ChunkByte takes each byte and divides it up into an array of 4 bit <b>pairs</b> (quaternary base) with the lowest pair first - i.e ChunkByte(1)={1,0,0,0} and ChunkByte(127)={3,3,3,1}. So, ChunkArray([6,127]) outputs [2, 1, 0, 0, 3, 3, 3, 1]. The reason for doing this is that it is then easy to turn the quaternary array into an array of any <b>even</b> bit length by specifying the number of bit-pairs to recombine. This is what reSlice does. It takes the quaternary array and the number of bits and channels and turns it into a series of numbers that addDeltaframe can process.
If this seems convoluted, it is!


Suggestions for improvements are more than welcome - Christof Schwiening cjs30@cam.ac.uk
