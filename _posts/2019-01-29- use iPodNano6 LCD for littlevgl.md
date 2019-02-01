This blog tells you something about **How to use the iPodNano6 LCD for LittlevGL.** I am going to hack the screen that is supposed to display on an Apple's iPodNano6 for LittlevGL with an Espressif ESP32 Wifi/BLE SoC.<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/Running_littlevGL.JPG)
LCD for iPod Nano6 uses MIPI Display Serial Interface (MIPI DSI) which is a high-speed serial interface between a host processor and a display module. LCDs belong to this category are very common for smartphones, tablets, and smartwatches. Reference is available from MIPI alliance page at https://www.mipi.org/specifications/dsi. Some MIPI LCDs on hands are shown here.<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/Some_mipi_displays.jpg)
*Googling* the keyword MIPI brings up several pdf documents of hundred pages. It is always fun to learn from specifications like this - http://bfiles.chinaaet.com/justlxy/blog/20171114/1000019445-6364627609238902374892404.pdf.
It states *"MIPI DSI specifies the interface between a host processor and a display..."* and finally a picture like this shows up that I can barely understand.
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/mipi_IF.jpg)<br>
If a 100-pages specification takes too much time, this may be all you need to know about MIPI D'PHY RX<br>
https://www.edn.com/Pdf/ViewPdf?contentItemId=4440302<br>
Transmission speed of MIPI is very high, ranging from 1.0Gbps/lane to 4.5Gbps/lane with 1-4 data lane plus 1 clock signal all in differential buses. Voltage swing driven by the difference buses is also different from RGB/MCU-typed LCD. For MIPI DSI there are high-speed (HS) and low-speed (LS) modes to drive 200mV peak-to-peak and 1.2V whereas data of RGB/MCU-typed LCDs is carried with single-ended signals matching VDDIO of MCU host.<br>
Table below summaries the difference.<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/mipi_vs_conventional-LCD.jpg)<br>

Usually interface of a MIPI LCD needs much less pins and lower voltage than its MCU/RGB counterpart. 
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/Pinout_compare.jpg)<br>

The problem is, how do we drive a MIPI display when there is no DSI output from our MCU (like ESP32) and how to port it to LittlevGL? Here comes the MIPI bridge IC - SSD2805, which is an interface chip to convert between RGB/8080 video signal to MIPI signal.<br> ![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/SSD2805_top.jpg)<br>
This is a very tiny chip of 5*5mm with 0.5mm pitch BGA!<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/SSD2805_bottom.jpg)

The block diagram of my setup is shown below.<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/block_diagram.jpg?raw=true)<br>
ESP32 is programmed with ESP-IDF (Espressif IoT Development Framework). Its installation procedure is described in full details at https://docs.espressif.com/projects/esp-idf/en/latest/get-started/index.html. My host computer is a Windows 7 Pro SP1 64-bit Operating System. Hardware is an old Intel Core i5 with 8GB RAM. I have followed the default installation path described in ESP-IDF's Getting Started Guide. It gave me back a mingw32.exe application under C:\msys32\.<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/mingw32_folders.jpg)<br>
At first I was not comfortable with command line tool like mingw32.exe. With innumerable Google searches I tried to install Eclipse IDE. Unfortunately all hours in Eclipse became futile. At the end I found the time spent on configuring Eclipse was even more than programming itself so I just gave it up. Don't mean Eclipse is bad. It is just me not able to get it work.<br>
Because there is no standard evaluation kit for ESP32 + SSD2805 + MIPI Display combo, I was forced to use jumper cables to wire up things with mess like this :(<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/messy_wireup.JPG)<br>
Boards employed include:<br>
(1) ESP32-Pico-Kit v4<br>
(2) SSD2805 breakout board Release 3 <br>
(3) 1.54 inch LH154Q01 MIPI display with CTP on PCB. SSD2541 CTP driver is soldered on this board.<br>
(4) Plus a lot jumper cables!<br>
The pinout diagram is illustrated below.<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/pinout_eps32_LCD.jpg)
To work with LittlevGL, the prerequisite is a fully working LCD and touch screen drivers outside it. I started with a program of 5 source files to drive the LCD listed below :<br>
```
(1) i2s_8080_hello_world.c
(2) SSD2805_8080_drv.c and .h
(3) i2s_lcd.c and .h
```
Source files i2s_lcd.c and .h were modified from their github source at :<br>
https://github.com/espressif/esp-iot-solution/tree/master/components/i2s_devices/lcd_common.<br>
ESP32 uses I2S module to write in 8080 8-bit parallel mode. DMA is used to queue command and data.<br>
Full source code of this project `i2s_8080_lcd`can be downloaded at the end of this page:<br>
To compile this project, just copy the complete folder to any place you find it convenient, in my case it is D:\esp32\i2s_8080_lcd. 
Launch mingw32.exe from C:/msys32<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/Launch_mingw32.jpg)<br>
Change directory to the root of Makefile with `cd D:/esp32/i2s_8080_lcd`<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/cd_i2s_8080_lcd_dir.jpg)<br>
Set the right serial port with `make menuconfig`<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/make_menuconfig.jpg)<br>
Browse to Serial flasher config ---> set it to COM2 (for my case).<br> 
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/COM2_to_use.jpg)<br>
Click EXIT several times and click `<Yes>` at the end to save new configurations.<br>
The last step is to `make flash`<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/make_flash.jpg)<br>
Now a fake AppleWatch is visible.<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/AppleWatch.JPG)<br>
Screen capture below shows all public functions of SSD2805. No text print, no shape draw or framebuffer operation. All GUI-related features are left to LittlevGL with a single API function `SSD2805_dispFlush(args)`, which has been designed to match the blueprint required not more or less. This is also the only function get called when screen refresh or update is required.<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/SSD2805_8080_drv_h.jpg)<br>

Similarly the driver for CTP was developed and tested with basic program that prints coordinates of finger with pressure to serial port. Screen capture of SSD2541.h is shown below. API function `SSD2541_getPoint(args)` is the only interface required by LittlevGL.<br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/SSD2541_h.jpg)<br>
In mingw32 console type `cd D:/esp32/SSd2541_drv_test`, repeat the same procedure as SSD2805 by `make menuconfig`, set Serial flasher config ---> to COM2 (in my case). Save changes and finally `make flash`. This time we need a terminal program like Serial Monitor of Arduino. Screen capture below shows a stream from Serial Monitor with finger released from (96,113), touched at (88,117) with varying pressure and then released again. LittlevGL requires that touch coordinates shoud be the last valid point that when the figner is released. <br>
![](https://github.com/techtoys/blog/blob/master/assets/iPodNano6/SSD2541_finger_event.jpg)<br>




 

