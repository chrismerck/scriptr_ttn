# TheThingsNetwork meets scriptr;

To realize the dream of the Internet of Things, we need an ubiquitous IoT data network. While smart dishwashers and smart garage doors can connect via a home's WiFi network, many IoT devices are out of range of existing wired or WiFi networks. Cellular and satellite networks have excellent coverage, but are costly in terms of airtime fees and power. The Internet of Things demands an outdoor, long-range, and low-power data network solution.

Recent developments in [LoRaWAN](https://www.lora-alliance.org/) radio technology allows low-power sensors to connect to distant internet gateways over free-to-use radio spectrum. Several competing commercial LoRaWAN providers now exist, or companies can build their own networks of gateways. However, the proliferation of different LoRaWAN networks in the same geographical area wastes resources and may cause crowding of the radio spectrum. Furthermore, commercial LoRaWAN providers charge for access to their networks just as with cellular and satellite.

The [TheThingsNetwork](http://thethingsnetwork.org/) (TTN) is an open-source data network owned and operated by its users built on top of LoRaWAN. Anyone is free to setup a gateway, and anyone may use those gateways at no charge. All source code for the network software is [developed in the open on GitHub](https://github.com/TheThingsNetwork) to harness the strengths of developers world-wide. TTN is currently in 50+ cities and is growing rapidly through the efforts of local communities.

In this tutorial, we will cover how to connect a device via TheThingsNetwork to Scriptr. We cover step-by-step how to connect a LoRa radio to a Raspberry Pi single-board computer. Also, as you may not have a gateway in range yet, we also cover gateway setup. We then show how with just a few lines of code, you can get TheThingsNetwork data into Scriptr.

## Overview

In this tutorial, we will use the following hardware:

* Raspberry Pi (a good [starter kit](https://www.canakit.com/raspberry-pi-starter-ultimate-kit.html) is available from CanaKit)
* USB Keyboard and Mouse for Raspberry Pi
* HDMI Monitor and Cable for Raspberry Pi
* [915MHz RN2903 LoRaMOTE from Microchip](http://www.microchip.com/DevelopmentTools/ProductDetails.aspx?PartNO=dm164139)
* [Multi-Tech Conduit Gateway](http://www.digikey.com/product-detail/en/multi-tech-systems/MTCDT-H5-210A-US-EU-GB/881-1237-ND/5246366)
* [Multi-Tech LoRa mCard](http://www.digikey.com/product-detail/en/multi-tech-systems/MTAC-LORA-915/881-1239-ND/5246368)
* [Multi-Tech 915MHz Antenna](http://www.digikey.com/product-detail/en/multi-tech-systems/AN868-915A-10HRA/881-1242-ND/5246371)

We will cover how to connect a Raspberry Pi node through the Multi-Tech Conduit gateway to TheThingsNetwork. Then we show how to pull data from TTN into Scriptr.

![block diagram](overview.png)

## Raspberry Pi

We will assume you have a Raspberry Pi or other Linux-based single-board computer to use as your node hardware. For initial setup you will need internet access. You may alternatively use a PC running Linux to act as the node for testing purposes, as it makes no difference as far as TheThingsNetwork and Scriptr are concerned.

![photo of raspberry pi](rpi.jpg)

### Download Python Script

First, connect to your Raspberry Pi and get to a bash command line. Then create a directory for source code as follows:

    cd
    mkdir src
    cd src

Now download `git` if needed, and use git to download the example node script from GitHub:

    sudo apt-get install git
    git clone https://github.com/chrismerck/ttn_node_py.git
    cd ttn_node_py

Now try running the `lora_mote_send.py` script with no arguments. You should get usage information:

    python lora_mote_send.py

### Attach Microchip LoRaMOTE

The Microchip LoRaMOTE is a development board for Microchip's RN2903 (or RN2483 in Europe) LoRaWAN tranceiver module. Many other development boards and tranceivers are available, however the RN2903 is one of the easiest to use. In addition to the RN2903 tranceiver, the LoRaMOTE has a built in PIC microcontroller, temperature and light sensors, AAA battery pack, and USB port. In this tutorial we will not use the sensors, but simply pass data through from the Raspberry Pi to a gateway.

![photo of loramote](lora_mote.jpg)

When you plug in the LoRaMOTE to the Raspberry Pi, Raspbian Linux will create a serial port device in its file system for communicating with the Mote. To tell the name of that serial device, you will want to first clear the device message log by running the following command at the Raspberry Pi's bash prompt:

    sudo dmesg -c

Next, plug in the LoRaMOTE to the Pi using the provided USB mini-B cable. Now again run the `dmesg` command. You should see a number of messages in the console like this:

    [198997.954943] usb 1-1.1: new full-speed USB device number 13 using xhci_hcd
    [198998.062393] usb 1-1.1: New USB device found, idVendor=04d8, idProduct=000a
    [198998.062406] usb 1-1.1: New USB device strings: Mfr=1, Product=2, SerialNumber=0
    [198998.062407] usb 1-1.1: Product: CDC RS-232 Emulation Demo
    [198998.062408] usb 1-1.1: Manufacturer: Microchip Technology Inc.
    [198998.063314] cdc_acm 1-1.1:1.0: ttyACM0: USB ACM device

The important line is the last one, where we learn that the device `/dev/ttyACM0` has been created in Linux to allow communicating with the LoRaMOTE.

Now try using `lora_mote_send.py` to send a test packet:

    python lora_mote_send.py /dev/ttyACM0 DEADBEEF AABBCCDD

Here you will need to replace `ttyACM0` with the name of the serial device on your Raspberry Pi, which may be different. The second argument `DEADBEEF` is the device address that the node will use. For now these are randomly assigned by users but will be assigned by the network in the future. The third argument, `AABBCCDD` is the data to send as exadecimal bytes. You should see the following:

    LoRa Tranceiver Configured.
    Sending message:  AABBCCDD
    Message sent successfully!

Note that although the message was successfully sent, LoRaWAN messages are unconfirmed by default, meaning that we get no confirmation from the gateway that the message was received.

## Gateway

In order to connect nodes to TheThingsNetwork and Scriptr, the nodes must be in range of at least one LoRaWAN gateway. The gateway receives the LoRa signal from the node and forwards the packet up to a ThingsNetwork router, after which the data is made available to applications such as one you will build in Scriptr. There are multiple gateways commercially available. We will use the MultiTech Conduit mLinux gateway with a LoRa mCard module.

![photo of conduit](conduit.jpg)

First you will need to assemble the gateway:

1. Install the LoRa mCard into the Conduit's AP1 slot.
2. Attach the 915MHz antenna to the LoRa mCard's RF port.
3. Attach the cellular antenna to the Conduit's CELL port.
4. Power the Conduit by plugging in the provided AC-DC converter.

### Logging into the Gateway

Next, connect the Conduit to your PC using the provided USB micro-B cable. If you are running OSX or Linux, use the `dmesg` procedure given in the previous section for determining the serial port for communicating with the Conduit. Then open a serial console with the following console command on OSX or Linux PC:

    screen /dev/ttyACM0 115200

Where you must replace `/dev/ttyACM0` with the serial port appropriate to your machine. The program `screen` is a simple serial terminal installed on most UNIX-like operating systems. You could use another serial terminal if you prefer, such as Hyperterminal on Windows systems. Upon connecting, you should be greeted with the following banner and login prompt:
        
![mlinux login](mlinux_login.png)

Enter the default username `root` and password `root`. You should be dropped to a root bash shell:

    root@mtcdt:~# 

### Check Firmware Version

Next you should check that you are running an up-to-date version of the Conduit's mLinux operating system. The latest version is listed on [MultiTech's website](http://www.multitech.net/developer/downloads/) and is 3.1.0 at the time of writing. The firmware version is found in the login banner or by running the following command at the Conduit's bash prompt:

    cat /etc/mlinux-version

If you have a version prior to 3.1.0, contact MultiTech for update instructions.

### Set the Time

Configure the time on the Conduit by running the following commands:

    ln -fs /usr/share/zoneinfo/America/New_York /etc/localtime 
    date "2010-04-02 14:58:01"
    hwclock -u -wm

Where you should replate `America/New_York` by your local timezone, and the time in the `date` command with the current date and time.

### Connect to the Internet

The Conduit supports connecting to the internet via ethernet or cellular. Here we will cover an Ethernet connection.

Plug the Conduit into an Ethernet network using an ethernet cable. If your network supports DHCP, then the Conduit should already be online. If static IP addressing is required, then edit the `/etc/network/interfaces` file:

    vi /etc/network/interfaces

Where you can replace `vi` with `nano` if you prefer.

Edit the `Wired interface` lines with your IP address, subnet, and gateway information. For static IP configuration, an additional `post-up` line is required to make DNS work. For example:

    # Wired interface
    auto eth0
    #iface eth0 inet dhcp
    iface eth0 inet static
    address 192.168.1.201
    netmask 255.255.255.0
    gateway 192.168.1.1
    post-up echo “nameserver 8.8.8.8″ > /etc/resolv.conf

If you edited the `interfaces` file for static IP, then run the following command to cause Linux to reload the configuration:

    /etc/init.d/networking restart

Now test the internet connection:

    ping 8.8.8.8

And with DNS:

    ping google.com

### Install Packet Forwarder

Next we will need to install a program called a "packet forwarder", which forwards all packets recieved over the air and pushes them up to a network server on the internet. Run the following command to download Kersing's `poly-packet-forwarder`:

    wget --no-check-certificate https://github.com/kersing/packet_forwarder/blob/master/multitech-bin/poly-packet-forwarder_2.1-r2_arm926ejste.ipk?raw=true

Then install the package by running:

    opkg install poly-packet-forwarder_2.1-r2_arm926ejste.ipk

### Edit Config Files

Next we will download pre-edited config files to control where packets are forwarded and which frequencies the Conduit is listening on.

At the Conduit's bash prompt, run the following commands:

    wget --no-check-certificate https://github.com/chrismerck/conduit_config/blob/master/etc/init.d/lora-network-server?raw=true -O /etc/init.d/lora-network-server
    wget --no-check-certificate https://github.com/chrismerck/conduit_config/blob/master/var/config/lora/global_conf.json?raw=true -O /var/config/lora/global_conf.json
    wget --no-check-certificate https://github.com/chrismerck/conduit_config/blob/master/var/config/lora/local_conf.json?raw=true -O /var/config/lora/local_conf.json
   
And now edit your local configuration using `vi` or your preferred editor:

    vi /var/config/lora/local_conf.json

Be sure to set your `gateway_ID` to a unique 64-bit number and set the `contact_email` and `description` fields. You should also put the latitude, longitude, and altitude (in meters) of the the gateway in the `ref_latitude`, `ref_longitude`, and `ref_altitude` fields, so that other TTN users can see where your gateway is located.

### Start the Packet Forwarder

Now try (re)starting the packet forwarder with the following command:

    /etc/init.d/lora-network-server restart

To see the log output, run the following:

    tail -f /var/log/lora-pkt-fwd-1.log

This will display data as it is written to the log file. Press CTRL-C to quit when finished observing the log. Wait about 30 seconds for the packet forwarder to start, and look for the following lines in the log:

    ### [UPSTREAM] ###
    # RF packets received by concentrator: 0
    # CRC_OK: 0.00%, CRC_FAIL: 0.00%, NO_CRC: 0.00%
    # RF packets forwarded: 0 (0 bytes)

This indicates that no packets were received yet by the gateway. Now, go to your Raspberry Pi console and send a packet again:

    python lora_mote_send.py /dev/ttyACM0 DEADBEEF AABBCCDD

In a moment, you should see in the Conduit log that a packet was demodulated and forwarded:

    # RF packets received by concentrator: 1
    # CRC_OK: 100.00%, CRC_FAIL: 0.00%, NO_CRC: 0.00%
    # RF packets forwarded: 1 (17 bytes)

This confirms that you successfully have the Raspberry Pi uploading data to the gateway via LoRa modulation.

## TheThingsNetwork

Next we will test reception of our packets by TheThingsNetwork. This step is optional. You will need a PC with a Linux operating system.

First, install the mosquitto MQTT clients:

    sudo apt-get install mosquitto-clients

Now subscribe to messages from your node:

    mosquitto_sub -h croft.thethings.girovito.nl -t nodes/DEADBEEF/packets

Next time you send a packet from your node, you should see a JSON packet such as:

    {"gatewayEui":"008000000000ABFF","nodeEui":"DEADBEEF","time":"2016-04-21T22:02:57.810580982Z",
     "frequency":912.1,"dataRate":"SF7BW125","rssi":-47,"snr":9.5,
     "rawData":"QO++rd4AAAAB5LOD2m4hSKE=","data":"3q2+7w=="}

The `data` field is encoded in Base64, and when decoded to hex reads `AABBCCDD`.

You can also subscribe to status messages from your gateway:

    mosquitto_sub -h croft.thethings.girovito.nl -t gateways/008000000000ABFF/status

Where you should replace `008000000000ABFF` with the gateway EUI you specified in `local_config.json` on the Conduit.

## Scriptr

Now that you have data from your Raspberry Pi pushing to TheThingsNetwork, 
we will write a Scriptr script to subscribe to packets from our gateway and nodes on TTN.

### MQTT Subscription

**NOTE**: This section will need to be adapted based on the syntax of the upcoming MQTT support in Scriptr.

Create a new script in your workspace called `ttn_demo`. We will write a few lines of Javascript to connect to TheThingsNetwork MQTT broker to receive packets for our node.

Start by creating an `mqtt` client object and connecting to TTN:

    /* mqtt init */ 
    var mqtt    = require('mqtt');
    var mqtt_client  = mqtt.connect('mqtt://croft.thethings.girovito.nl');

Next we subscribe to the MQTT "topics" corresponding to our node's ID:

    /* subscribe to some topics */
    mqtt_client.on('connect', function () {
      mqtt_client.subscribe('nodes/DEADBEEF/packets');
      /* additional nodes may be specified here */
    });


    /* forward MQTT packets to REST */
    mqtt_client.on('message', function (topic, message) {
      console.log("MQTT Received Packet from TTN: ",message.toString());
    });

When you run the script and generate some packets from your node, you should see the JSON packets dumped to the Scriptr console.

## What's Next?

Now that you have a ThingsNetwork node linked into Scriptr, you can develop applications that go beyond the reach of WiFi, without paying for expensive cellular or satellite data plans.

Do be aware that this setup uses a common session key,
 which means that although packets are technically encrypted over the air,
 the packets can be easily decrypted because the encryption key is published.
 See [TheThingsNetwork WiKi](http://thethingsnetwork.org/wiki/) for documentation
 on setting up per-node encryption keys.

## Further Reading

* [TheThingsNetwork WiKi](http://thethingsnetwork.org/wiki/)
* [LoRaWAN specification](https://www.lora-alliance.org/Contact/Request-Specification-Form)
* [MQTT protocol specification](http://mqtt.org/documentation)







