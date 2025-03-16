***************************
DCC-EX Turntable Controller
***************************

This is a controller for DCC-EX turntables, operating as a native DCC-EX protocol client to utilise a turntable object and allow a user to operate it.

Turntable operations are displayed on a round LCD, with a rotary encoder utilised for user input.

It is capable of operating any type of turntable configured using the new DCC-EX turntable object, including both EX-Turntable and DCC turntables.

Information on the new turntable object is available on the `DCC-EX website <https://dcc-ex.com/ex-commandstation/accessories/turntables/index.html>`_.

This can be connected to an EX-CommandStation via either a serial connection, or WiFi if using an ESP32 WROOM based microcontroller.

How it works (and DCC-EX turntable configuration)
=================================================

When creating a turntable object in DCC-EX EX-CommandStation, you can specify an angle for each position, which is what this DCC-EX Turntable Controller software uses to configure the turntable display.

The angle of the home position is taken from what would be 12 o'clock on a traditional clock face (the top centre of the circle), with each position's angle taken from the home position in a clockwise direction.

.. warning:: 

  If you do not specify an angle for each position when creating a turntable object, this software will be unable to correctly render each position on screen.

Setting up a turntable object
-----------------------------

Turntable objects can be defined interactively via the ``<I ...>`` commands, or in EXRAIL via the ``DCC_TURNTABLE()``, ``EXTT_TURNTABLE()``, and ``TT_ADDPOSITION()`` commands. This documentation will focus only on EXRAIL and, in particular, an EX-Turntable object definition as an example.

.. note:: 

  To allow for angles with a single decimal place, the DCC-EX turntable object's valid angles are from 0 - 3600, so this software divides the angle by 10 to obtain the correct angle in degrees.

This is the example EX-Turntable definition in the DCC-EX documentation, updated with angles to draw the home and other positions on screen in the correct locations.

Home is at the bottom of the display (180 degrees from the top of the screen).

The roundhouse tracks are separated by 10 degrees:

- Stall 1 is 80 degrees from home
- Stall 2 is 90 degrees from home
- Stall 3 is 100 degrees from home
- Connection to the rest of the layout is 270 degrees from home
- To reverse into stall 1, a position is created at 260 degrees from home
- To reverse into stall 3, a position is created at 280 degrees from home

This would be configured in EXRAIL via myAutomation.h as such:

.. code-block:: c++

  HAL(EXTurntable,600,1,0x60)
  EXTT_TURNTABLE(1,600,1800,"My EX-Turntable")
  TT_ADDPOSITION(1,1,114,800,"Stall 1")
  TT_ADDPOSITION(1,2,227,900,"Stall 2")
  TT_ADDPOSITION(1,3,341,1000,"Stall 3")
  TT_ADDPOSITION(1,4,2159,2600,"Rev stall 1")
  TT_ADDPOSITION(1,5,2273,2700,"Layout")
  TT_ADDPOSITION(1,6,2386,2800,"Rev stall 3")

As you can see, setting the turntable object up in this manner means you only need to do it once, and this turntable controller software needs no further configuration, meaning any updates to your turntable object will automatically be reflected on the display without any further configuration effort.

Operation
---------

When the software starts, it will retrieve the first configured turntable object from the EX-CommandStation, and display the various positions on screen, with the turntable bridge represented at the current position of the turntable object.

To operate the turntable, simply rotate the rotary encoder to the desired position, and a single click of the encoder's button will send the selection to the CommandStation.

If the turntable is configured as an EX-Turntable, the display will flash the bridge and position name while the turntable is moving. As there is no feedback from DCC turntables, this will not happen with those.

To select the home position (again, only valid for EX-Turntable), either select it using the rotary encoder and click the button, or just double click it.

Holding down the rotary encoder's button will reset the DCC-EX Turntable Controller.

Software
========

The software for the turntable controller is available in my `DCC-EX Turntable Controller <https://github.com/peteGSX-Projects/DCCEXTurntableController>`_ GitHub repository.

There is some documentation generated for the code itself in `GitHub pages <https://petegsx-projects.github.io/DCCEXTurntableController/>`_.

Installation
------------

The repository is setup to enable PlatformIO to be used with VSCode and will automatically install the required libraries:

- DCC-EX/DCCEXProtocol
- bodmer/TFT_eSPI
- avandalen/Switch

The platformio.ini file is written to configure the TFT_eSPI library correctly according to the :ref:`dccex-turntable-controller/index:pins and connections` section below, meaning manual configuration of the TFT_eSPI library shouldn't be required.

If you prefer, you can also use the Arduino IDE, which means you will need to:

- Install the above libraries
- Configure the TFT_eSPI manually according to the `library's documentation <https://github.com/Bodmer/TFT_eSPI>`_

There is plenty of information and tutorials available to help with VSCode, PlatformIO, the Arduino IDE, and libraries, so I will not cover that here.

Configuration
-------------

To get up and running, just uploading the software to the Blackpill or ESP32 and connecting the display, rotary encoder, and serial connection should simply work "out of the box".

If you wish to customise the colours and font in use, you can copy the included "myConfig.example.h" to "myConfig.h" and edit it to suit. Instructions are in the file.

If you wish to use WiFi on the ESP32, you will need to configure the following parameters as outlined in "myConfig.h":

- WIFI_SSID
- WIFI_PASSWORD
- COMMANDSTATION_IP
- COMMANDSTATION_PORT

You will also need to set the ``CLIENT_TYPE`` to ``WIFI_CLIENT``:

.. code-block:: 

  #define CLIENT_TYPE WIFI_CLIENT

Hardware requirements
=====================

To assemble the DCC-EX Turntable Controller, you will need:

- An STM32F411CEU6 Blackpill microcontroller or Espressif ESP32 WROOM based microcontroller
- A rotary encoder such as the KY-040
- A GC9A01 based round LCD

Further to this, you will need a serial connection to a DCC-EX CommandStation unless using WiFi.

Pins and connections
--------------------

The below pins are used on to connect to the CommandStation's serial port, rotary encoder, and GC9A01 LCD.

Remember to cross Rx/Tx so that the Blackpill Rx connects to the CommandStation's Tx, and vice versa. Also be sure to connect a common ground, and take into account if the CommandStation uses 3.3V or 5V logic, as the Blackpill is a 3.3V uC.

.. list-table:: 
  :widths: auto
  :header-rows: 1

  * - Device Connection
    - ESP32 Dev Kit Pin
    - Blackpill Pin
  * - Rotary encoder DT
    - 26
    - PB14
  * - Rotary encoder CLK
    - 27
    - PB13
  * - Rotary encoder button
    - 25
    - PB15
  * - GC9A01 DIN
    - 23
    - PA6
  * - GC9A01 CLK
    - 18
    - PA5
  * - GC9A01 CS
    - 15
    - PA4
  * - GC9A01 DC
    - 2
    - PA3
  * - GC9A01 RST
    - 4
    - PA2
  * - GC9A01 BL
    - 21
    - PA1
  * - Serial Tx
    - 17
    - PA9
  * - Serial Rx
    - 16
    - PA10
