# Lsc-solar-camera-tinker_project
In this repository i will share what i have found when opening the newely released lsc 1080p solar camera and i will try to dump its firmware and maybe even add a firmware you can directly burn onto the flash chip

Please note this is just for fun and its not intended for malicious use or using my firmware image for malicious purposes and i also dont support that! Also i wont be responsible if you brick your device! IF you are the manufacturer of this device and wish to have this removed feel free to email me to 309Electronics@gmail.com and i will answer and remove the repo if it violates any laws or copyright i am happy to cooperate.

with that out of the way now i will tell you about the project.
So recently i decided to go to a store called Action which is in the netherlands and germany and itss a store that sells a lot of diferent kind of products like house hold, electronics, consumables, toys, garden, kitchen etc etc at cheap prices. And i always go through the electronics section and thats where i saw this new product. Lsc is a brand Action sells that is owned by electrocirkel BV in the netherlands and makes lighting and iot devices. I managed to see this product which i never saw before and it seemed new and cool to tinker with so i bought it and it costed 37,95 euro. I first paired it to the app and after that i tore it down. First challenge was opening it up but i quickly found a screw neatly hidden under the "waterproof" rubber sealing that seals the ports and buttons on the bottom. i unscrewed it and opened it 

![IMG20240405204246](https://github.com/RX309Electronics/Lsc-solar-camera-tinker_project/assets/114357631/30715905-8020-493a-9cb5-4a730b3f65ec)
This image shows the product box.


![IMG20240405204356](https://github.com/RX309Electronics/Lsc-solar-camera-tinker_project/assets/114357631/5bce3bdd-f19b-40bc-a4bf-6bda1f9fc7eb)
![IMG20240405204350](https://github.com/RX309Electronics/Lsc-solar-camera-tinker_project/assets/114357631/7cc7b516-69b7-4c06-9cfc-cb9599c9783c)
This is the front and back side.


![IMG20240405204411](https://github.com/RX309Electronics/Lsc-solar-camera-tinker_project/assets/114357631/b7fb34d2-a296-431c-88de-989d1d944475)
![IMG20240405204402](https://github.com/RX309Electronics/Lsc-solar-camera-tinker_project/assets/114357631/f362a0d4-fcc2-4b70-9a15-7c63939ee06b)
The waterproof sealing and where i found the screw and the buttons.


![IMG20240405204511](https://github.com/RX309Electronics/Lsc-solar-camera-tinker_project/assets/114357631/067385e1-bab2-4ee0-bedf-a6e99f4bacd5)
This is the product torn down.


![IMG20240405204559](https://github.com/RX309Electronics/Lsc-solar-camera-tinker_project/assets/114357631/043e1ddb-e373-44b8-96fa-8ead36223585)
The board of the device.


![IMG20240405204630](https://github.com/RX309Electronics/Lsc-solar-camera-tinker_project/assets/114357631/4e0f71f4-22af-4476-883b-c91f75bbf9d0)
The board showing the Ingenic Semiconductor T31 Soc which actually has a mips core running at 1.5ghz and a additional low power riscV core running at 500mhz. The Mips runs linux and the riscV is used as a general purpose microcontroller handeling the motion detection capabilities and housekeeping the system. It also wakes up the Mips core from sleep/shutdown when motion is detected because the mips actually shuts off after a few seconds to save power while the riscV is always on. The riscV also comminucates with the mips. Close to the soc is the flash chip made by xmc its the 25qh128c model which is 3.3volt and is a 128mbit (PLEASE NOTE IT SAYS MEGABIT NOT MEGABYTE) Spi flash memory that holds the Uboot bootloader, Linux kernel, filesystem and Tuya application stack that handles all communication with the tuya cloud services and the lsc app (which is basically tuya's smartlife app under the hood). The wifi module is a azurewave aw-nm372sm incorporating a Broadcom (now product line taken over by cypress semiconductor) cyw43438_a1 chipset.
![Screenshot_2024-04-05-21-15-33-29_4a24d271e133915ae237d4bec6ffe368](https://github.com/RX309Electronics/Lsc-solar-camera-tinker_project/assets/114357631/597c9342-05f8-47c4-82f6-60e2866e8bd1)
Block diagram of the ingenic soc.


![IMG20240405211726](https://github.com/RX309Electronics/Lsc-solar-camera-tinker_project/assets/114357631/aa1a4e0e-440a-4b58-ae16-d3f22ec3cefa)
Uart interface of the Linux os neatly labelled (i soldered a wire to it already!) Please note i could not get the top uart to talk to me so maybe its disabled but i think its the bootloader uart. It uses uboot as its bootloader but idk how to get it to talk to me and i sadly could not enter the bootloader shell of the device but only the Linux Os uart port. 


![image](https://github.com/RX309Electronics/Lsc-solar-camera-tinker_project/assets/114357631/11be442c-83b7-4088-827c-0ae063f12bb2)
Next i hooked up my trusty cp2102 usb to uart converter to my pc and the devices working serial port and saw lots of bootmessages come through (its quite chatty and to my surprise i saw the wifi name and password printed out as spare text yikes!) (its in a file called wiif.cfg) I tried to login but its a bit messy/impossible because sometimes the messages get in the way and also the os did not seem to have a normal login prompt and every password i tried failed but it does not need a password as i discovered later when i went and dumped the firmware that the shadow file did not seem to have a password hash in it. It seems to call the tuya servers a lot of times and all their api's. The wifi part did scare me a bit though!. the camera does not have any open ports and the only way to get acces to its internal os is via uart. No matter what keys i bashed on boot it would just boot and later not let me log in or it would poweroff to save power because no motion detected which shuts down the mips to save power which is kind of anoying and not really avoidable. 
Putty output is also posted but i removed all mentions of my password and wifi ssid so i dont dox myself and replaced them with YOURSSID and YOURPASSWD.


![IMG20240405214435](https://github.com/RX309Electronics/Lsc-solar-camera-tinker_project/assets/114357631/345a1de1-40bb-4e51-a02e-4654b57aad76)
Reading the firmware of the device via a ch341 connected via an soic8 clip to the board. I could not find a working reset so i just shorted the 1.8v line to gnd which actually disabled the mips core from interfering with the firmware reading. After reading i verified the firmware image and now i have a .bin file. I actually did reset the camera to factory before i dumped it so no mentions of my wifi network wuld be included because again i dont want to dox myself so hopefully all credentials are gone fingers crossed... 

I actually used binwalk to extract the firmware and saw a jffs filesystem and it actually has linux kernel 3.10 (at least mine did) in the binary i saw the bootloader arguments and it seems to run linuxrc as the init option and changing it in the binary (by using the inbuilt search option of the neoprogrammer software) to init=/bin/sh managed to make it so i now can acces the busybox shell without getting interupted by the tuya stack although sadly the poweroff timer is still there and the riscV shuts off the cpu after a few seconds but just move in front of it every few seconds to keep it awake. You can still start the tuya stack by executing ./linuxrc in the shell but this will not give you back the shell unles you reboot the device again. 
Inspecting the rcS script in /etc/init.d/ it seems that the telnet daemon is actually listed as an option that it enables but its commented out sadly. Although i think you can unpack the filesystem and edit the rcS script and then repack it into a binary and flash it on the device via the ch341 clip (someone already did this but without the ch341 soic clip). 

Thats it for now!
This is a project i am currently active on and you will see some further updates once i know more and maybe upload the custom firmware with telnet enabled and a removed sleep timer so stay tuned!


