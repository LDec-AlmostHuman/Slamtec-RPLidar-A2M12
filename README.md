After trying Slamtec manual for running RPLidar A2M12, I run into many issues.

I have not tested it on MacOS (Silicone) and not tested on any Windows flavors.

Below is a fix for Ubuntu 20.04 & Ubuntu 22.04.

I have tested it on Jetson NX with Ubuntu 20.04 and AMD PC with both Ubuntu 20.04, Ubuntu 22.04 and Xubuntu 20.04.
On Ubuntu 20.04, below works without major problems.

On Ubuntu 22.04, after getting
```
git clone https://github.com/Slamtec/rplidar_sdk.git
```
you will need to correct one line somewhere around line 170 in rplidar_sdk/sdk/src/arch/linux/net_socket.cpp

Replace:
```
return ans<=0?RESULT_OPERATION_FAIL:RESULT_OK;
```
with:
```
return ans == nullptr ? RESULT_OPERATION_FAIL : RESULT_OK;
```
save the file, and **get back** to rplidar_sdk directory and now should compile.
```
make
```
On fresh Xubuntu 20.04 you might need to install below packages:
```
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y make g++ gcc git gedit
```
RPLidar A2M12, has 256000 communication speed, so you will need to change the speed to 256000 with the switch on the device.
Check if you can see CP2102 USB to UART Bridge Controller by running:
```
usb-devices
```
It should return something like this, that means that the driver is loaded. If not, look a few lines below for installation instructions from Slamtec and after start from the beginning.
```
T:  Bus=05 Lev=01 Prnt=01 Port=02 Cnt=01 Dev#=  4 Spd=12  MxCh= 0
D:  Ver= 1.10 Cls=00(>ifc ) Sub=00 Prot=00 MxPS=64 #Cfgs=  1
P:  Vendor=10c4 ProdID=ea60 Rev=01.00
S:  Manufacturer=Silicon Labs
S:  Product=CP2102 USB to UART Bridge Controller
S:  SerialNumber=0001
C:  #Ifs= 1 Cfg#= 1 Atr=80 MxPwr=100mA
I:  If#= 0 Alt= 0 #EPs= 2 Cls=ff(vend.) Sub=00 Prot=00 Driver=cp210x
E:  Ad=01(O) Atr=02(Bulk) MxPS=  64 Ivl=0ms
E:  Ad=81(I) Atr=02(Bulk) MxPS=  64 Ivl=0ms
```
Run below line to see ownership and which ttyUSBx you see
```
ls -l /dev | grep ttyUSB 
```
It should return something like this:
```
crw-rw----   1 root  dialout 188,     0 Mar 11 06:44 ttyUSB0
```
Change permissions to that device (others are suggesting using 666, but for me only 777 worked):
```
sudo chmod 666 /dev/ttyUSB0
```
or
```
sudo chmod 777 /dev/ttyUSB0
```
Run again:
```
ls -l /dev | grep ttyUSB
```
Now should return changed permission:
```
crwxrwxrwx   1 root  dialout 188,     0 Mar 11 06:44 ttyUSB0
```
Add current user to dialout:
```
sudo adduser $USER dialout
```
Installation of drivers from Slamtec, this is in case if usd-devices doesn't return any info on Product=CP2102 USB to UART Bridge Controller. After you will need to start the above procedure from the beginning.
```
git clone https://github.com/Slamtec/rplidar_sdk.git
cd rplidar_sdk
```
Install ament_cmake if you don't have one installed already - my installation was missing ament-cmake and was throwing errors.
```
sudo apt-get update
sudo apt-get -y install ament-cmake 
```
now run
```
make
```
If no errors you can run what was compiled:
```
cd output/Linux/Release/
```
Now test what is the actual speed of the interface. It will work correctly not throwing errors if we check this part. We will use that value later:
```
./custom_baudrate  /dev/ttyUSB0 256000
```
In return you should get something like below.
```
Baudrate negotiation demo for SLAMTEC LIDAR.
Try to establish communication to the LIDAR using the baudrate at /dev/ttyUSB0@256000 ...
The baudrate detected by the pair is 257175 bps. Error: 0.459 %
Wow, we just communicate with the LIDAR using a non-standard baudrate : 256000!.
SLAMTEC LIDAR S/N: (32 characters serial number)
Firmware Ver: 1.32
Hardware Rev: 6
```
As you can see it throws an Error: 0.459 % in my case.
Baudrate is 257175 bps in my case, so we will correct the line.
Please use **your baudrate** for all below tests.
```
./custom_baudrate /dev/ttyUSB0 257175
```
Now it returned without any errors:
```
Baudrate negotiation demo for SLAMTEC LIDAR.
Try to establish communication to the LIDAR using the baudrate at /dev/ttyUSB0@257175 ...
The baudrate detected by the pair is 257175 bps. Error: 0.000 %
Wow, we just communicate with the LIDAR using a non-standard baudrate : 257175!.
SLAMTEC LIDAR S/N: (32 characters serial number)
Firmware Ver: 1.32
Hardware Rev: 6
```
Now let's test simple_grabber:
```
./simple_grabber --channel --serial /dev/ttyUSB0 257175
```
It should return something like this:
```
Lidar health status : OK. (errorcode: 0)
waiting for data...
                                                                           
                                                                           
                                                                           
                                                                           
                                                                           
                                                                           
                                                                           
                                                                           
                                                                           
                                                                           
                                                                           
                                                                           
                                                                           
                                 *       *                                 
                       * * *     *       *                                 
                       * * *     *       *                                 
                       * * *     *       *                                 
                       *** **    *       *                                 
                       *** **    *       * *                               
***************************************************************************
---------------------------------------------------------------------------
Do you want to see all the data? (y/n)
```
For more data you can press y and will show some more data and will stop.
To stop press n and will stop.

Now let's test ultra_simple
```
./ultra_simple --channel --serial /dev/ttyUSB0 257175
```
It should return something like this:
```
theta: 0.00 Dist: 00000.00 Q: 0 
theta: 0.23 Dist: 00000.00 Q: 0 
theta: 0.47 Dist: 00000.00 Q: 0 
theta: 0.70 Dist: 00000.00 Q: 0 
theta: 0.93 Dist: 00000.00 Q: 0 
theta: 1.17 Dist: 00000.00 Q: 0 
theta: 1.40 Dist: 00000.00 Q: 0 
....
theta: 358.59 Dist: 00000.00 Q: 0 
theta: 358.83 Dist: 00000.00 Q: 0 
theta: 359.06 Dist: 00000.00 Q: 0 
theta: 359.30 Dist: 00000.00 Q: 0 
theta: 359.53 Dist: 00000.00 Q: 0 
theta: 359.76 Dist: 00000.00 Q: 0 
```
To stop CTRL - C
I hope all works:).

I will post as well corrected python files for git clone https://github.com/Slamtec/sllidar_ros2.git
