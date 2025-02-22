ESP32 door bell to sip call
===========================

On startup the application associates with the compiled in wlan access point and
registers on the SIP server.

The project also contains a http server to perform firmware updates by uploading the firmware bin file.

Once a signal is detected on the selected GPIO, a call is initiated to a target number. On the phone, the custom string is displayed.
After the configured timeout is elapsed, the call is canceled. If the signal is detected again, before the timer is elapsed, the timer
is started again.

Tested with:

* AVM Fritzbox 7390
* AVM Fritzbox 7490 (Firmware 7.28)
* AVM Fritzbox 7590
* local FreeSWITCH installation

Programming
-----------

The source code is mixed C and C++.

This application is to be used with `Espressif IoT Development Framework`_ (ESP-IDF). It is tested with version v5.0 (rev 7f5ecbe533b2666df6b10658a023b5f637320696)

Please check ESP-IDF docs for getting started instructions.


Building
++++++++

The project now uses cmake, so after initializing your environment with the relevant variables from esp-idf you can use idf.py to build, flash etc::

  cd <this project's root dir>
  idf.py menuconfig
  idf.py build
  idf.py flash monitor

To build for another ESP32 soc, e.g. the ESP32C3::

  cd <this project's root dir>
  idf.py set-target esp32c3
  idf.py menuconfig
  idf.py build
  idf.py flash monitor

See `Selecting soc build target`_ for more details.


To build this project for the pc (linux, e.g. ubuntu or fedora), a sample (not all features are supported, yet)::

  mkdir <build dir>
  cd <build dir>
  cmake <this project's root dir>/native
  make

The sip server configuration must be done in the defines of the file <this project's root dir>/native/main.cpp.

The following libraries are required for this (e.g. on fedora)::

  sudo dnf install asio-devel mbedtls-devel

Code formatting
+++++++++++++++

Clang-format is used to format the code. The settings are stored in .clangformat. The format of external files, e.g.
components/sip_client/include/boost/sml.hpp should not be changed. To run clang-format (e.g. version 12) over all files::

  find . -regex '.*\.\(cpp\|cc\|cxx\|h\)' -exec clang-format -style=file -i {} \;

Hardware
--------

An ESP32 board can be used. Only one external GPIO (input is sufficient) must be available, to detect the call trigger.
To test this, two PC817 opto coupler are used to detect the AC signal (about 12V from the bell transformer). The input diodes of the opto couplers are connected in parallel and opposing directions.
In series, a 2k Resistor is used. This may have to be tweaked according to the input voltage.
The output transistors of the opto couplers are connected in parallel in the same polarity to pull the signal to ground, if a current flows through one of the input diodes. A pull up resistor (either internal in the ESP32 or external) must be used to pull the signal to 3V3 if no input current is detected and the output transistors are switched off.

Instead of two PC817 opto couplers, one PC814 can be used to detect the AC signal. Because of the different CTR, the resistor values must be tweaked.
If it is sufficient to only detect one half-wave of the AC signal (this is normally the case) one PC817 opto coupler and a simple diode (e.g. 1N4148) is sufficient. The diode ensures that the voltage of the input diode of the opto coupler is not above the threshold. The 1N4148 must be connected anti-parallel to the input diode of the PC817.

.. image:: hw/door_bell_input_schematic.svg
	   :width: 600pt


If the bell transformer delivers enough power, the ESP32 can be powered from it. A bridge rectifier, a big capacitor and a cheap switching regulator board can be used for that.


License
-------

If not otherwise specified, code in this repository is Copyright (C) 2017-2021 Christian Taedcke <hacking@taedcke.com>, licensed under the Apache License 2.0 as described in the file LICENSE.

Misc Information
----------------

On the AVM Fritzbox the number \*\*9 can be used to let all connected phones ring.


.. _`Espressif IoT Development Framework`: https://esp-idf.readthedocs.io/
.. _`Selecting soc build target`: https://docs.espressif.com/projects/esp-idf/en/v5.0/esp32/api-guides/tools/idf-py.html#select-the-target-chip-set-target
