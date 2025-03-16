**********************
DCC-EX Simple Throttle
**********************

.. sidebar::

  .. contents:: On this page
      :depth: 2
      :local:

To accompany my mimic panel's serial throttle (see :doc:`/tri-throttle/index`), I decided a very simple WiFi enabled, palm-sized throttle I could walk around the layout with would be a very handy addition, hence the creation of this DCC-EX Simple Throttle.

Note that there is an option to run this on an STM32F103C8 Bluepill via a serial connection instead.

.. image:: /_static/images/simple-throttle/dccex-simple-throttle.png
  :alt: DCC-EX Simple Throttle
  :scale: 30%

Hardware
========

The hardware required is pretty simple, consisting of a Wemos D1 Mini or Lolin32 Lite (which incorporates a LiPO battery connector), a LiPO battery, rotary encoder (such as the KY-040), and an SH1106 compatible OLED (either SPI or I2C).

I opted for a 1.3" OLED for ease of readability, as it's still small enough to fit in the palm of my hand when assembled with the rest of the hardware.

I have designed and 3D printed a case to suit the specific components which is available on `Thingiverse <https://www.thingiverse.com/thing:6980561>`_.

I neglected to take any assembly photos, but this is a look inside the completed throttle, noting that the battery is mounted first underneath the Lolin32 Lite board.

.. image:: /_static/images/simple-throttle/assembly.png
  :alt: Assembly Inside
  :scale: 20%

Component connections
---------------------

This table outlines the pins in use to connect to the various components.

.. list-table:: 
  :widths: auto
  :header-rows: 1

  * - Device Connection
    - Wemos/Lolin32 Pin
    - Bluepill Pin
  * - Rotary encoder DT
    - 12
    - PC14
  * - Rotary encoder CLK
    - 14
    - PC15
  * - Rotary encoder button
    - 13
    - PA0
  * - OLED SDA (I2C)
    - 23
    - PB7
  * - OLED SCL (I2C)
    - 22
    - PB6
  * - OLED MOSI (SPI)
    - 23
    - PA7
  * - OLED MISO (SPI)
    - 19
    - PA6
  * - OLED CLK (SPI)
    - 18
    - PA5
  * - OLED CS (SPI)
    - 5
    - PA4
  * - OLED DC (SPI)
    - 2
    - PA3
  * - Serial Tx
    - N/A
    - PA10
  * - Serial Rx
    - N/A
    - PA9

Software
========

The software is located in my `DCCEXSimpleThrottle <https://github.com/peteGSX-Projects/DCCEXSimpleThrottle>`_ GitHub repository.

This uses the DCCEXProtocol Arduino library to interact with an EX-CommandStation using the native DCC-EX protocol rather than the WiThrottle protocol, ensuring future-proof feature compatibility with DCC-EX software updates.

The code documentation is available using `this link <https://petegsx-projects.github.io/DCCEXSimpleThrottle/index.html>`_.

Installation
------------

The software is configured to use VSCode with PlatformIO and will therefore ensure the correct libraries are installed when compiling and loading the software onto your device.

If using the Arduino IDE, you will need to ensure platform support for STMicroelectronics STM32 and/or Espressif is installed.

Further, you will need to ensure these libraries are installed:

- DCC-EX/DCCEXProtocol
- olikraus/U8g2

There is plenty of information available on the Internet at large around how to accomplish installing these items, so I won't cover that here.

Configuration
-------------

Once the hardware components are connected according to the defaults in :ref:`simple-throttle/index:component connections`, for the Bluepill platform, no further configuration should be required.

When using the Wemos D1 Mini/Lolin32 Lite, you will need to configure the WiFi credentials and EX-CommandStation IP address and port details for each EX-CommandStation you wish to connect to. Defining multiple sets of details is supported, meaning you can take this to your club or friend's place without having to reconfigure it every time.

If for some reason you need to use different pin connections, these can also be adjusted to suit.

All user configuration changes can be made by copying the provided "myConfig.example.h" file to "myConfig.h" (note this file name is case sensitive, and Windows users beware an inadvertent ".txt" extension being added).

Instructions are included in the file for updating the various details, but the WiFi/EX-CommandStation options are outlined here for clarity as well.

Configuring WiFi and EX-CommandStation details
----------------------------------------------

Set ``COMMANDSTATION_COUNT`` to the exact number of EX-CommandStations you will set up connection details for.

For each of the parameters, you must provide the exact same number of elements that you defined for ``COMMANDSTATION_COUNT`` as shown in the example configuration provided. Further to this, every element aside from the port must be surrounded by quotes ``"<element>"``, and each list of elements must be surrounded by curly brackets ``{<list>}``.

When configuring the parameters:

- COMMANDSTATION_NAMES is a list of descriptive names to help you select the appropriate one from the menu
- COMMANDSTATION_IPS is a list of the IP addresses for the CommandStations, one for each configured CommandStation (even if the same IP is used for multiple, each one must be specified)
- COMMANDSTATION_PORTS is a list of the ports associated with the CommandStations (quite likely these will all be the default 2560)
- COMMANDSTATION_SSIDS is a list of the WiFi SSIDs required to connect to the CommandStations
- COMMANDSTATION_PASSWORDS is a list of WiFi passwords associated with the SSIDs to connect to the CommandStations

.. code-block:: c++

  /*
  Define WiFi connection parameters here
  These are irrelevant if using Bluepill and a serial connection
  */
  #define COMMANDSTATION_COUNT 2 // The number of EX-CommandStations to define

  /*
  Define connection options for each EX-CommandStation entry

  Each line must have the exact same number of items as set in COMMANDSTATION_COUNT above
  Each element must be surrounded by quotes "" and separated by a commad
  Each list of elements must be surrounded by curly brackets {}

  #define COMMANDSTATION_NAMES {"Example 1", "Example 2"}
  #define COMMANDSTATION_IPS {"192.168.4.1", "192.168.0.1"}
  #define COMMANDSTATION_PORTS {2560, 2560}
  #define COMMANDSTATION_SSIDS {"SSID1", "SSID2"}
  #define COMMANDSTATION_PASSWORDS {"Password1", "Password2"}
  */
  #define COMMANDSTATION_NAMES { "CommandStation 1", "CommandStation 2" }
  #define COMMANDSTATION_IPS { "192.168.4.1", "192.168.0.1" }
  #define COMMANDSTATION_PORTS { 2560, 2560 }
  #define COMMANDSTATION_SSIDS { "SSID1", "SSID2" }
  #define COMMANDSTATION_PASSWORDS { "Password1", "Password2" }

Configuring a local roster
--------------------------

If you wish, you can configure a local roster of Locos that will be added to the ``Select Loco`` menu, and each of these items will be prefaced with ``*`` to indicate they are not retrieved from your EX-CommandStation roster.

Similar to setting up the WiFi connections above, set ``LOCAL_ROSTER_COUNT`` to the exact number of local roster entries you are adding, and add the exact same number of entries to:

- LOCAL_ROSTER_NAMES which is the friendly name you will see in the ``Select Loco`` menu
- LOCAL_ROSTER_ADDRESSES which is the DCC accress of the Loco

.. code-block:: c++

  #define LOCAL_ROSTER_COUNT 3

  #define LOCAL_ROSTER_NAMES {"Local Loco 1001", "Local Loco 1002", "Local Loco 1003"}
  #define LOCAL_ROSTER_ADDRESSES {1001, 1002, 1003}

Operation
=========

When navigating menus, scroll up and down with the rotary encoder, and click the rotary encoder's button to select the highlighted item. Depending on the current context, the rotary encoder button can be double clicked or held to perform different actions.

During operation, there are three contexts on screen to switch through:

- ``Select server`` menu (not shown when using Bluepill with a serial connection)
- ``Select loco`` menu
- ``Select action`` menu
- Throttle screen

"Select server" menu
--------------------

When starting, you will first see the "Select server" menu (except on Bluepill with a serial connection). Select one of the EX-CommandStation entries configured in the :ref:`simple-throttle/index:configuring wifi and ex-commandstation details` section to connect to it.

Once connected, the :ref:`simple-throttle/index:"select loco" menu` will be displayed.

"Select loco" menu
------------------

.. image:: /_static/images/simple-throttle/select-loco.png
  :alt: Select Loco menu
  :scale: 30%

Providing a roster of locomotives has been configured in the connected EX-CommandStation, selecting one from the menu will allow you to control it.

If you have configured a local roster as per :ref:`simple-throttle/index:configuring a local roster`, then these will be available in this menu, prefaced by a ``*``.

- Select a Loco using the rotary encoder to scroll through the list, and press the button once to select a Loco
- Double clicking the rotary encoder button will navigate to the :ref:`simple-throttle/index:"select action" menu`
- Holding the rotary encoder button down will cause the EX-CommandStation to attempt to read the Loco address from the programming track

Once either a Loco is selected, or the programming track is read, the :ref:`simple-throttle/index:throttle screen` will be displayed. If reading the Loco on the programming track fails, you will be returned to the ``Select loco`` menu.

"Select action" menu
--------------------

.. image:: /_static/images/simple-throttle/select-action.png
  :alt: Select Action menu
  :scale: 30%

This menu allows you to perform various track power related actions.

- Single click the rotary encoder button to select an option
- Double click the rotary encoder button to return to the :ref:`simple-throttle/index:"select loco" menu`

- Join PROG Track - Joins the programming track to the main track, allowing a Loco to be operated on it
- Power Main On - Turns power on to the Main track only
- Power Main Off - Turns power off on the Main track only
- Power Prog On - Turns power on to the Prog track only
- Power Prog Off - Turns power off on the Prog track only

Throttle screen
-------------------

.. image:: /_static/images/simple-throttle/throttle-screen.png
  :alt: Throttle Screen
  :scale: 30%

While on this screen, rotating the rotary encoder will increase or decrease the locomotive speed.

If speed is greater than zero:

- Single click the rotary encoder button to stop the selected locomotive
- Double click the rotary encoder button to toggle function 0 on and off, which is typically the headlight
- Hold the rotary encoder button for longer than half a second to trigger an emergency stop

If speed is zero:

- Single click the rotary encoder button to change direction
- Double click the rotary encoder to display the :ref:`simple-throttle/index:"select loco" menu`
- Hold the rotary encoder button for longer than half a second to toggle track power on or off
