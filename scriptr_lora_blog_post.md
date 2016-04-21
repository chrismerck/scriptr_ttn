# TheThingsNetwork meets scriptr;

intro

## Overview

block diagram

hardware required

## Raspberry Pi

login to raspberry pi (ssh, console, or keyboard+monitor)

sudo apt-get install git
cd
mkdir src
cd src
git clone X
cd X

dmesg -c
connect loramote
dmesg -c
note serial device
python send_packet.py serial data

note: for now this uses a common session key,
 which means that although packets are encrypted over the air,
 the packets can be easily decrypted because the encryption key is published.
 if you run your own ttn handler you can specify your own
 session key to achieve encryption over-the-air and on-the-wire.


## Gateway

check & update firmware

install poly packet forwarder

download default config, add UID, config for subband 7

test that packets are forwarding with tail -f log
look for number of packets received with good CRC > 0

next test by using mosquitto_sub directly to public ttn handler
note: this will be replaced in a few months with a new URL

## Scriptr

we will write a scriptr script to subscribe to packets from our gateway and nodes on TTN.

once packets are received, we unpack the data

can test by return values (?) -- we need mqtt packets to trigger a script running, not an HTTP request...

now we push it to InitialState for visualization

## InitialState

ed todo

nice screen cap




