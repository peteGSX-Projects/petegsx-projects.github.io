********************************
DCC-EX Rotary Encoder (archived)
********************************

.. warning:: 

  This repository has now been marked as a public archive as the project has been superceded by the new :doc:`/dccex-turntable-controller/index` project instead.

  If you're looking to control a DCC-EX turntable with this software, I highly recommend looking at the DCC-EX Turntable Controller instead, as this project will receive no further development.

.. sidebar::

  .. contents:: On this page
    :depth: 2
    :local:

This software has been written to enable an Arduino with a rotary encoder connected to it to be integrated with a DCC-EX CommandStation.

The primary purpose is to select a position to control EX-Turntable via EX-RAIL automation, however it could be used for other purposes.

There are currently two different modes for the encoder, with either option being suitable for use in a mimic panel:

1. As a generic control knob with a standard, SPI connected monochrome OLED display to select a position between -127 and 127.
2. As a turntable controller utilising an SPI connected GC9A01 round colour LCD display to select predefined positions.

The EX-CommandStation device driver has the capability of sending feedback to the Arduino when a turntable move has been completed. See :ref:`rotary-encoder/ex-rail-integration:receiving feedback from the ex-commandstation` for details.

In addition, it is also possible to have the EX-CommmandStation notify when a turntable move has been initiated via an EXRAIL automation rather than directly by this rotary encoder software. See :ref:`rotary-encoder/ex-rail-integration:receiving a new position from the ex-commandstation` for details.

.. note:: 

  One other option tested was to use the rotary encoder with a pointer type knob (without a display) to select positions printed directly on a mimic panel. However, rotating the knob to specific positions has proven unreliable and this idea has been abandoned. It's possible that it relates to the quality of rotary encoder used or the way the software has been written, so if anyone else is interested in testing/implementing this feature then you are more than welcome to do so, and if you wish, you can get in contact with me (see :doc:`/contact`). I have left the code intact that allows re-aligning the encoder with a specific "home" position (see :ref:`rotary-encoder/overview:usage` below).

.. toctree:: 
  :hidden:
  :maxdepth: 1

  control-knob
  turntable-controller
  ex-rail-integration

Software requirements
=====================

To utilise the rotary encoder, you must be running version 5.0.0 (or later) of EX-CommandStation.

`Get it here <https://dcc-ex.com>`_.

The latest version of the rotary encoder software is available in my `DCC-EX Rotary Encoder <https://github.com/peteGSX-Projects/DCCEXRotaryEncoder>`_ repository.

Refer to :doc:`/rotary-encoder/ex-rail-integration` and the `DCC-EX EX-Turntable documentation <https://dcc-ex.com/ex-turntable/test-and-tune.html#controlling-ex-turntable-with-a-rotary-encoder>`_ for further information on how this interacts with an EX-CommandStation.

In "Turntable Controller" mode, the GC9A01 round LCD requires the `Arduino_GFX library <https://github.com/moononournation/Arduino_GFX>`_.

.. note:: 

  For convenience, all required libraries have been included in the repository with the software to ensure there are no issues with different versions etc.

Hardware requirements
=====================

This requires an AVR based Arduino (tested on a Nano), a rotary encoder (tested with a KY-040), and an SPI connected OLED or GC9A01 round LCD display.

See the relevant :doc:`/rotary-encoder/control-knob` or :doc:`/rotary-encoder/turntable-controller` page for hardware and connection details.

An ESP32 can also be used, but note that this has only been tested using a Wroom Dev Kit with GC9A01 round LCD.

The STM32F411CEU6 Blackpill and STM32F103C8 Bluepill are also supported.

Refer to the config.example.h file to see which pins are in use for the ESP32, Blackpill, and Bluepill devices.

Also, a suitable DCC-EX EX-CommandStation must be available, with the rotary encoder Arduino connected to it via I2C.

Refer to the `DCC-EX documentation <https://dcc-ex.com>`_ for further information on EX-CommandStation.

.. warning:: 

  For those wishing to use Blackpill and Bluepill, the included PlatformIO configuration relies on using an STLink device to upload the software, and enables the USB port for serial input/output and will not support uploading the software.

  My recommendation is save your sanity and stick with STLink, I won't be supporting other methods with the software. In addition, Bluepill users may experience USB connectivity issues due to an incorrect resistor on clone boards. There is plenty of info on the Internet about this so I won't go into the details.

  I would highly recommend spending the very small amount of extra money to stick with the Blackpill.

Installation and configuration
==============================

The provided "config.example.h" file in addition to the Fritzing diagrams included on the relevant pages should help identify the correct pins to connect the encoder and display to.

Various configuration options can be edited in the file "config.h". By default, this file does not exist, and an example file "config.example.h" with the various options has been provided. It is recommended to copy the example file to your own "config.h", as the example file will be overwritten by any future changes to the software.

General configuration options
-----------------------------

.. list-table:: 
  :widths: auto
  :header-rows: 1

  * - Option
    - Default
    - Details and options
  * - I2C_ADDRESS
    - 0x78
    - Can be any valid and available I2C address
  * - MODE
    - TURNTABLE
    - Select either KNOB or TURNTABLE, defines the operating mode
  * - DIAG
    - Commented out/disabled
    - Uncomment for continuous output of encoder position to the serial console
  * - FEEDBACK
    - Uncommented/enabled
    - Comment out to disable receiving feedback from the EX-CommandStation device driver
  * - POSITION
    - Uncommented/enabled
    - Comment out to disable receiving position updates from the EX-CommandStation device driver

.. code-block:: cpp

  /////////////////////////////////////////////////////////////////////////////////////
  //  START: General configuration options.
  /////////////////////////////////////////////////////////////////////////////////////
  #define I2C_ADDRESS 0x78  // Default 0x78, can be any valid, available I2C address
  #define MODE TURNTABLE    // Default TURNTABLE
  // #define MODE KNOB
  // #define DIAG           // Uncomment to enable continous output of encoder position
  #define FEEDBACK          // Comment out to disable feedback from CommandStation
  #define POSITION          // Comment out to disable receiving position updates from CommandStation
  /////////////////////////////////////////////////////////////////////////////////////
  //  END: General configuration options.
  /////////////////////////////////////////////////////////////////////////////////////

Rotary encoder options
----------------------

.. list-table:: 
  :widths: auto
  :header-rows: 1

  * - Option
    - Default
    - Details and options
  * - HALF_STEP
    - Uncommented/enabled
    - Comment out this line to enter into full step mode
  * - POLARITY
    - 0
    - Set to 0 for clockwise to increment, counter-clockwise to decrement, set to 1 to reverse direction
  * - DEBOUNCE
    - 50
    - Debounce delay in milliseconds, increase or decrease as necessary to reduce false button presses
  * - LONG_PRESS
    - 1000
    - Time in milliseconds to detect a long press of the button
  * - ENABLE_PULLUPS
    - Uncommented/enabled
    - Comment out if your encoder button does not require pullups enabled
  * - ROTARY_BTN
    - 2
    - Pin the encoder button connects to
  * - ROTARY_DT
    - 5
    - Pin the encoder DT pin connects to
  * - ROTARY_CLK
    - 6
    - Pin the encoder clock/CLK pin connects to
  * - DIR_NONE
    - 0x0
    - Hex value returned by the rotary encoder process command when no change detected
  * - DIR_CW
    - 0x10
    - Hex value returned by the rotary encoder process command when a clockwise change is detected
  * - DIR_CCW
    - 0x20
    - Hex value returned by the rotary encoder process command when a counter-clockwise change is detected

.. code-block:: cpp

  /////////////////////////////////////////////////////////////////////////////////////
  //  START: Rotary encoder configuration options.
  /////////////////////////////////////////////////////////////////////////////////////
  #define HALF_STEP         // Comment out to use full step mode
  #define POLARITY 0        // Set to 1 to reverse rotation direction
  #define DEBOUNCE 50       // Adjust if necessary to prevent false button presses
  #define LONG_PRESS 1000   // Adjust if necessary for long press detection
  #define ENABLE_PULLUPS    // Comment out if input does not require pull up
  #define ROTARY_BTN 2      // Define encoder button pin
  #define ROTARY_DT 5       // Define encoder DT pin
  #define ROTARY_CLK 6      // Define encoder clock pin

  // Values returned by 'process', these should not need modification.
  // No complete step yet.
  #define DIR_NONE 0x0
  // Clockwise step.
  #define DIR_CW 0x10
  // Anti-clockwise step.
  #define DIR_CCW 0x20
  /////////////////////////////////////////////////////////////////////////////////////
  //  END: Rotary encoder configuration options.
  /////////////////////////////////////////////////////////////////////////////////////

Display options
---------------

For the OLED configuration options avaiable in control knob mode, refer to :ref:`rotary-encoder/control-knob:knob mode configuration options`, and for the GC9A01 round LCD configuration options available in turntable mode, refer to :ref:`rotary-encoder/turntable-controller:turntable mode configuration options`.

Usage
=====

Once the software has been configured and uploaded to the rotary encoder Arduino, and your EX-CommandStation has been configured to support it with the updated software uploaded, you can simply rotate the encoder to the appropriate position desired, and a push of the button will send that position to the EX-CommandStation.

In the generic control knob mode, it will send a position between -127 and 127. The OLED will display the last selected position in addition to the position reflected by the current knob position.

In turntable controller mode, the knob rotates a representative turntable on the GC9A01 round LCD, and will send the predefined position number when the representative turntable is aligned with a position, and the encoder button is pushed. 

As per the introductory note at the top of this page, functionality has been retained to rotate the knob back to a "home" position. This is accomplished by holding the button down for a second then releasing it, resulting in the OLED display indicating it can now be re-homed. Simply rotate the encoder to the desired home or 0 position, then press the button once to confirm. Normal functionality will then resume. While this functionality still operates in turntable controller mode, there is no visual indication on the GC9A01 round LCD.