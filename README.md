# TwinCAT3_w_ADS_Sum_RdWr
A sample application with the ADS Sum Read/Write function

Guide to ADS Sum Commands.

+ In the BECKHOFF ADS specificatoin, basically interface is assigned a certain  Index Group.
+ For example, PLC interface will have Index Group 0x4020 for its internal variables, and the NC interface has 0x4001 for   Axis 1, and so on. The data within the Index Group is then specified by the offset. 
+ To do the SUM Commands, you will need to execute the AdsReadWrite command.

For sum-read:
+ Essentially, what is happening is that you are sending to the router a list of list of variable it should find the values of. The location is described by the variable's Index Group, and Index Group Offset, and data size.
+ In the AdsStream being sent to the router, the format is basically:
[Index Group of Var 1][Offset of Var 1][Size of Var 1]...[Index Group of Var n][Offset of Var n][Size of Var n]
+ These 3 data per variable are all int32 type, therefore the total stream size is actually (n * 12)
+ It will then return to you (in the form of an AdsStream) the corresponding number of data of error codes, then the corresponding number of data you had sent previously. Which is
[Error code 1][Error code 2]...[Error code n][Data 1][Data 2]...[Data n]
+ The error codes are int32, and the sequence is according to the variables you have written previously
+ The total stream size is therefore (n * 4) + (size of data 1 + size of data 2 + ... + size of data n)

For sum-write:
+ The principle is the same as sum-read.
+ The main difference is that you are now sending the variable information AND the data to be written into the PLC
+ In the AdsStream being sent to the router, the format is basically:
[Index Group of Var 1][Offset of Var 1][Size of Var 1]...[Index Group of Var n][Offset of Var n][Size of Var n][Data 1][Data 2]...[Data n]
+ Therefore the total stream size is actually (n * 12) + (size of data 1 + size of data 2 + ... + size of data n)
+ It will then return to you (in the form of an AdsStream) the corresponding number of data of error codes
[Error code 1][Error code 2]...[Error code n]
+ The error codes are int32, and the sequence is according to the variables you have written previously
+ The total stream size is therefore (n * 4)

As a guide to the flow of doing a Sum-read command:
1) Connect the client, eg. tcClient.Connect(851)
2) For each PLC variable you want to read, create its variable handles
3) Create an AdsStream(n * 12) for sending to the router
4) Write into AdsStream the index group for the first variable, then the index group offset, then the data size
5) Write into AdsStream the same format for the next variables until done
6) Create an AdsStream((n*4) + (S1 + S2 + ... + Sn)) to receive the values
7) Use the ReadWrite command with both AdsStreams
8) ReadInt32 from the AdsStream n times to get the error codes for each variables
9) Finally, Readback the data in the corresponding sequence for the actual data

As a guide to the flow of doing a Sum-write command:
1) Connect the client, eg. tcClient.Connect(851)
2) For each PLC variable you want to read, create its variable handles
3) Create an AdsStream((n * 12) + (total size of data to be transferred)) for sending to the router
4) Write into AdsStream the index group for the first variable, then the index group offset, then the data size
5) Write into AdsStream the same format for the next variables until done
4) Write into the AdsStream, the corresponding sequence of data that are to be sent to the PLC
6) Create an AdsStream(n*4) to receive the values
7) Use the ReadWrite command with both AdsStreams
8) ReadInt32 the AdsStream from step 6 n times to know the error code for each send
