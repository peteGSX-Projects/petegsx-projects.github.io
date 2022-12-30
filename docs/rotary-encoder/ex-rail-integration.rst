*************************************
Device Driver and EX-RAIL Integration
*************************************

.. sidebar::

  .. contents:: On this page
    :depth: 2
    :local:

Enable the EX-CommandStation device driver
==========================================

To integrate the rotary encoder with your EX-CommandStation, ensure you are running the `Latest EX-CommandStation Unreleased Development Version <https://dcc-ex.com/download/ex-commandstation.html#latest-ex-commandstation-unreleased-development-version>`_ , which includes both the "IO_RotaryEncoder.h" device driver and EX-RAIL commands required.

The default I2C address for this device is 0x70, and you will need to enable the device driver via myHal.cpp:

Example:

.. code-block:: cpp
  
  #include "IO_RotaryEncoder.h"

  void halSetup() {
    RotaryEncoder::create(700, 1, 0x70);
  }

Refer to :doc:`/rotary-encoder/ex-rail-integration` and the `DCC-EX documentation <https://dcc-ex.com/ex-turntable/test-and-tune.html#controlling-ex-turntable-with-a-rotary-encoder>`_ for further information on how this interacts with an EX-CommandStation.

Receiving feedback from the EX-CommandStation
---------------------------------------------

The device driver included with EX-CommandStation is capable of sending feedback to this rotary encoder software to indicate whether a turntable is moving (1), or if the move has completed (0).

In turntable controller mode, the display will indicate the turntable is moving by blinking the representative turntable on and off, and will stop blinking when it is no longer moving.

Enabling feedback in the device driver
--------------------------------------

To enable feedback in the device driver, you simply need to specify to use two Vpins rather than one, and sending a ``SET(vpin)`` (moving started) or ``RESET(vpin)`` (moving complete) via your EX-RAIL automation will tell the device driver moving has started or completed. See :ref:`rotary-encoder/ex-rail-integration:turntable controller example with feedback` for an example of how this can be implemented.

.. code-block:: cpp

  void halSetup() {
    RotaryEncoder::create(700, 1, 0x70);       // Defining 1 Vpin disables feedback
  }

  void halSetup() {
    RotaryEncoder::create(700, 2, 0x70);      // Defining 2 Vpins enables feedback
  }

EX-RAIL automation
==================

Two EX-RAIL commands have been added to enable usage:

ONCHANGE() Event Handler
------------------------

An event handler has been added ``ONCHANGE(vpin)`` similar to the turnout and accessory event handlers.

This enables any position change of the rotary encoder to be used to define an activity.

IFRE() Test Statement
--------------------- 

A test statement ``IFRE(vpin, position)`` has been added to check for the specific position selected by the encoder.

Valid positions are from -127 to 127 in control knob mode, or 0 to 255 in turntable controller mode.

Control knob example
--------------------

This is a brief example of how to use the encoder in control knob mode to select some turntable positions, based on the myEX-Turntable.example.h file included with the CommandStation code:

.. code-block:: 

  // EX-Turntable macro and route definitions
  #define EX_TURNTABLE(route_id, reserve_id, vpin, steps, activity, desc) \
    ROUTE(route_id, desc) \
      RESERVE(reserve_id) \
      MOVETT(vpin, steps, activity) \
      WAITFOR(vpin) \
      FREE(reserve_id) \
      DONE

  EX_TURNTABLE(TTRoute1, Turntable, 600, 114, Turn, "Position 1")
  EX_TURNTABLE(TTRoute2, Turntable, 600, 227, Turn, "Position 2")
  EX_TURNTABLE(TTRoute3, Turntable, 600, 341, Turn, "Position 3")
  EX_TURNTABLE(TTRoute4, Turntable, 600, 2159, Turn, "Position 4")
  EX_TURNTABLE(TTRoute5, Turntable, 600, 2273, Turn, "Position 5")
  EX_TURNTABLE(TTRoute6, Turntable, 600, 2386, Turn, "Position 6")
  EX_TURNTABLE(TTRoute7, Turntable, 600, 0, Home, "Home turntable")

  // Rotary encoder event handler to select positions:
  ONCHANGE(700)
    IFRE(700, 1)
      START(TTRoute1)
    ENDIF
    IFRE(700, 2)
      START(TTRoute2)
    ENDIF
    IFRE(700, 3)
      START(TTRoute3)
    ENDIF
    IFRE(700, -1)
      START(TTRoute4)
    ENDIF
    IFRE(700, -2)
      START(TTRoute5)
    ENDIF
    IFRE(700, -3)
      START(TTRoute6)
    ENDIF
    IFRE(700, 0)
      START(TTRoute7)
    ENDIF
  DONE

  // Pre-defined aliases to ensure unique IDs are used.
  // Turntable reserve ID, valid is 0 - 255
  ALIAS(Turntable, 255)

  // Turntable ROUTE ID reservations, using <? TTRouteX> for uniqueness:
  ALIAS(TTRoute1)
  ALIAS(TTRoute2)
  ALIAS(TTRoute3)
  ALIAS(TTRoute4)
  ALIAS(TTRoute5)
  ALIAS(TTRoute6)
  ALIAS(TTRoute7)

Turntable controller example
----------------------------

This is a brief example of how to use the encoder in turntable controller mode to select some turntable positions, based on the myEX-Turntable.example.h file included with the CommandStation code:

.. code-block:: 

  // EX-Turntable macro and route definitions
  #define EX_TURNTABLE(route_id, reserve_id, vpin, steps, activity, desc) \
    ROUTE(route_id, desc) \
      RESERVE(reserve_id) \
      MOVETT(vpin, steps, activity) \
      WAITFOR(vpin) \
      FREE(reserve_id) \
      DONE

  EX_TURNTABLE(TTRoute1, Turntable, 600, 114, Turn, "Position 1")
  EX_TURNTABLE(TTRoute2, Turntable, 600, 227, Turn, "Position 2")
  EX_TURNTABLE(TTRoute3, Turntable, 600, 341, Turn, "Position 3")
  EX_TURNTABLE(TTRoute4, Turntable, 600, 2159, Turn, "Position 4")
  EX_TURNTABLE(TTRoute5, Turntable, 600, 2273, Turn, "Position 5")
  EX_TURNTABLE(TTRoute6, Turntable, 600, 2386, Turn, "Position 6")
  EX_TURNTABLE(TTRoute7, Turntable, 600, 0, Home, "Home turntable")

  // Rotary encoder event handler to select positions:
  ONCHANGE(700)
    IFRE(700, 1)
      START(TTRoute1)
    ENDIF
    IFRE(700, 2)
      START(TTRoute2)
    ENDIF
    IFRE(700, 3)
      START(TTRoute3)
    ENDIF
    IFRE(700, 4)
      START(TTRoute4)
    ENDIF
    IFRE(700, 5)
      START(TTRoute5)
    ENDIF
    IFRE(700, 6)
      START(TTRoute6)
    ENDIF
    IFRE(700, 0)
      START(TTRoute7)
    ENDIF
  DONE

  // Pre-defined aliases to ensure unique IDs are used.
  // Turntable reserve ID, valid is 0 - 255
  ALIAS(Turntable, 255)

  // Turntable ROUTE ID reservations, using <? TTRouteX> for uniqueness:
  ALIAS(TTRoute1)
  ALIAS(TTRoute2)
  ALIAS(TTRoute3)
  ALIAS(TTRoute4)
  ALIAS(TTRoute5)
  ALIAS(TTRoute6)
  ALIAS(TTRoute7)

Turntable controller example with feedback
------------------------------------------

This is a brief example of how to use the encoder in turntable controller mode to select some turntable positions, based on the myEX-Turntable.example.h file included with the CommandStation code.

Note the addition of the parameter "feedback_vpin" in the "EX_TURNTABLE" macro defining the second VPin for the rotary encoder, where the ``SET(feedback_vpin)`` sends feedback that the move has started, and the ``RESET(feedback_vpin)`` sends feedback that the move has completed.

.. code-block:: 

  // EX-Turntable macro and route definitions
  #define EX_TURNTABLE(route_id, reserve_id, vpin, steps, activity, desc, feedback_vpin) \
    ROUTE(route_id, desc) \
      RESERVE(reserve_id) \
      MOVETT(vpin, steps, activity) \
      SET(feedback_vpin) \
      WAITFOR(vpin) \
      RESET(feedback_vpin) \
      FREE(reserve_id) \
      DONE

  EX_TURNTABLE(TTRoute1, Turntable, 600, 114, Turn, "Position 1", 701)
  EX_TURNTABLE(TTRoute2, Turntable, 600, 227, Turn, "Position 2", 701)
  EX_TURNTABLE(TTRoute3, Turntable, 600, 341, Turn, "Position 3", 701)
  EX_TURNTABLE(TTRoute4, Turntable, 600, 2159, Turn, "Position 4", 701)
  EX_TURNTABLE(TTRoute5, Turntable, 600, 2273, Turn, "Position 5", 701)
  EX_TURNTABLE(TTRoute6, Turntable, 600, 2386, Turn, "Position 6", 701)
  EX_TURNTABLE(TTRoute7, Turntable, 600, 0, Home, "Home turntable", 701)

  // Rotary encoder event handler to select positions:
  ONCHANGE(700)
    IFRE(700, 1)
      START(TTRoute1)
    ENDIF
    IFRE(700, 2)
      START(TTRoute2)
    ENDIF
    IFRE(700, 3)
      START(TTRoute3)
    ENDIF
    IFRE(700, 4)
      START(TTRoute4)
    ENDIF
    IFRE(700, 5)
      START(TTRoute5)
    ENDIF
    IFRE(700, 6)
      START(TTRoute6)
    ENDIF
    IFRE(700, 0)
      START(TTRoute7)
    ENDIF
  DONE

  // Pre-defined aliases to ensure unique IDs are used.
  // Turntable reserve ID, valid is 0 - 255
  ALIAS(Turntable, 255)

  // Turntable ROUTE ID reservations, using <? TTRouteX> for uniqueness:
  ALIAS(TTRoute1)
  ALIAS(TTRoute2)
  ALIAS(TTRoute3)
  ALIAS(TTRoute4)
  ALIAS(TTRoute5)
  ALIAS(TTRoute6)
  ALIAS(TTRoute7)