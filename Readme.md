# Developing and flashing the firmware for temprature sensor.

The board used in order to implement the task is Iotlab-m3 with (STM32F013RE MCU). In this task pressure and temprature sensor is udes.
The temprature sensor used here is the inbuilt LPS331AP sensor. It also calculate the pressure.
In order to check more about sensor you can check the link https://www.iot-lab.info/docs/boards/iot-lab-m3/#sensors
During this task Grenoble architecture is used. 



## Flashing the Firmware on sensor nodes

In this task two nodes are used. The numbering of the node depends on the availability.

### 1. Initializing RIOT
``````
iotlab-experiment submit  -n "TempMeasurement" -d 120 -l 1,archi=m3:at86rf231+site=grenoble
``````
Must include this command if find error:
``````
source /opt/riot.source
``````
### 2. Flashing of the temp sensor nodes
``````   
cd Sensor/
make IOTLAB_NODE=auto flash
make IOTLAB_NODE=auto -C . term
``````

### 3. Submitting for node 2
  for node 2
``````
  iotlab-experiment submit -n TempMeasurement -d 120 -l grenoble,m3,2,./bin/iotlab-m3/sensors.elf
``````
### Press Enter for Sensor Reading.
``````
> usage: lps <temperature|pressure|start/stop>
> lps
  
> lps temperature start
``````


### 4. Coap Client and Server node initialization:

#### 1. Flashing Coap firmware
```
cd Coap/
make
```

#### 2. Submission of the firmware to two different node:
``````
iotlab-experiment submit -n Coap -d 60 -l grenoble,m3,3,./bin/iotlab-m3/Coap.elf

iotlab-experiment submit -n Coap -d 60 -l grenoble,m3,4,./bin/iotlab-m3/Coap.elf

nc m3-4 20000 - Server

ifconfig
``````
This command will give us the IPV6 address of the server.

```
nc m3-3 20000 - client
```
Here, a communication link will be check:
```````
 ping <server_ip>
 coap get fe80::a808:e073:a0df:945f /.well-known/core
 coap get fe80::a808:e073:a0df:945f 5683 /temperature
```````

# IPv6 Networking
using Riot examples
``````
cd RIOT/examples
``````
## Gnrc_networking firmware
For getting Channel and Pan ID
``````
cd RIOT/examples
```````
python
```````
import os,binascii,random
pan_id = binascii.b2a_hex(os.urandom(2)).decode()
channel = random.randint(11, 26)
print('Use CHANNEL={}, PAN_ID=0x{}'.format(channel, pan_id))
```````
Change it according to the channel and pan ID
``````
make -C gnrc_networking BOARD=iotlab-m3 DEFAULT_CHANNEL=14 DEFAULT_PAN_ID=0x845b
``````
## Flashing of IPv6 networking 
Two nodes for IPv6 flashing. (Nodes depends on availabibility)
``````
iotlab-experiment submit -n Ipv6 -d 60 -l grenoble,m3,5,gnrc_networking/bin/iotlab-m3/gnrc_networking.elf
iotlab-experiment submit -n Ipv6 -d 60 -l grenoble,m3,6,gnrc_networking/bin/iotlab-m3/gnrc_networking.elf
``````

# Border router Setting

Change it according to yours:
``````
make ETHOS_BAUDRATE=500000 DEFAULT_CHANNEL=14 PAN_ID=0x845b BOARD=iotlab-m3  -C gnrc_border_router clean all
``````
## Submission ofBorder Router firmware:
``````
iotlab-experiment submit -n Border_Router -d 60 -l grenoble,m3,7,gnrc_border_router/bin/iotlab-m3/gnrc_border_router.elf
``````

## IPv6 Setup - 
``````
nc m3-5 20000
```````

```````
ifconfig
```````

nc m3-6 20000
``````
``````
ifconfig
``````

## We then apply the ethos_uhcpd to border router node

``````
sudo ethos_uhcpd.py m3-7 tap0 2001:660:3207:04c1::1/64
``````


Here's the link of the uploaded video on youtube: 
https://www.youtube.com/watch?v=2O6R2AHsGgo&t=628s