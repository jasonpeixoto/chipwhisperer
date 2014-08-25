.. _tutorialsasebowcard:

Tutorial #B10: Using with SASEBO-W CardOS
=========================================

**NOTE**: This tutorial requires you are using at least version 0.04.


Background
----------

A number of extremely useful tools for side channel analysis are distributed by Morita Tech Co., Ltd under the `SAKURA <http://satoh.cs.uec.ac.jp/SAKURA/index.html>`_
project name. In fact much the original ChipWhisperer system was developed on these tools, and a great debt is owed to Akashi Satoh for this development.
 
 
This tutorial will demonstrate how the ChipWhisperer system can be used in tandem with the SASEBO-W system. The first part of this tutorial will target
the same ATMegaCard used by the SASEBO-W, and the second part of the tutorial will demonstrate how the ATMega328p can be connected to the SASEBO-W using
the interposer board.

Hardware Setup
--------------

About the ATMega Card
^^^^^^^^^^^^^^^^^^^^^

The ATMega Card is shown below:

.. image:: /images/tutorials/basic/scard/megacard.jpg
  :width: 400

This card contains an Atmel ATMega163 die along with a 24C256 EEPROM. You can see the internal pinout of these cards online. It should be noted that this card is
in a SmartCard *form factor*, but is essentially just a very old microcontroller (AtMega163). If you are unable to find this card but still wish to perform these
experiments, there are two other options:

 1. Purchase an ATMega16, which can be programmed with the ATMega163 binary (.hex file). See an `Atmel App-Note <http://www.atmel.com/Images/doc2517.pdf>`_
    on the subject. The AtMega16 *will not* fit on the Multi-Target board, meaning you must build your own board. You can then connect the appropriate
    IO lines to the SmartCard interface.
    
 2. Rebuild your code for the ATMega328p. This should require minimal changes to the source code, but note you cannot program a .hex file for a Mega163 into
    a Mega328P directly. You will need the complete source code.

Alternatively of course you can package your target algorithm into something like the demo SimpleSerial project too. There is no real need to use the SmartCard APDU
format, and the interface tends to be much slower on the ChipWhisperer system.

Programming
^^^^^^^^^^^

You will need an image to program into the SmartCard. This tutorial uses the SASEBO-W Card OS. Details of this are available from the
`SASEBO-W Page <http://satoh.cs.uec.ac.jp/SAKURA/hardware/SASEBO-W.html>`_. Download the file entitled 
*Smartcard sample binary for ATMega 163*, which is described in the document entitled *SASEBO-W Smart Card OS Specification Ver. 0.4-5*.

The first file will have a .hex inside it, which you must program using AVRStudio or similar. To use the built-in programmer, 
the following connections should be set:

 1. Remove all jumpers from the AVR and XMEGA sections of the MultiTarget board.
 
 2. Remove the AtMega328p from the socket.
 
 3. Set the oscillator for *3.579 MHz* (JP18), and set the *CLKOSC* jumper (JP17).
 
 4. Mount all four jumpers on the *AVR-PROG* section (JP8).
 
 5. Shunt both the *GND* and *VCC* resistors, as the programming will fail with those resistors in the power lines (JP7).

The following image shows these connections:

.. image:: /images/tutorials/basic/scard/programming.jpg

Then use AVRStudio to program the .hex file. The instructions for doing this are as in :ref:`buildprogrammingavr`, however when selecting
the AVR type select *ATMega163* instead of *ATMega328p*. Check the *Read Signature* option works, if not double-check the above hardware
connections.

Hardware setup for using Card Socket
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following describes the jumper settings when using the SmartCard socket on the MultiTarget Victim Board:

   1. NO jumpers mounted in XMEGA Portion or AVR Portion, ideally remove the AVR Chip as well
   2. 3.3V IO Level (JP20 set to *INT*.)
   3. The *3.579 MHz* oscillator is selected as the CLKOSC source (JP18)
   4. The *CLKOSC* is connected to the SmartCard Clock Network, along with connected to the *FPGAIN* pin (JP4)
   5. Trigger is selected as *AX2* (JP22)
   6. Power measurement taken from VCC shunt (JP7)
   7. Jumpers removed from the AVR-PROG header (JP8)
   
The following image shows this setup:

.. image:: /images/tutorials/basic/scard/attacksettings.jpg

Connect the 20-pin cable and SMA cable if not already connected, and plug your programmed MegaCard into the SmartCard socket. This completes
the hardware setup when using the card socket.

Hardware Setup using ATMega16
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

TODO

Hardware Setup using ATMega328p
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

TODO

Software Setup and Example Capture
----------------------------------

 1. Run the ChipWhisperer Capture software
 
 2. Download the CW Firmware (*Tools --> Download CW Firmware*), ensure the board is detected
 
 3. Select the following options on the *General Settings* tab:
    
  a. Scope Module: *ChipWhisperer/OpenADC*
  b. Target Module: *Smart Card*
  c. Trace Format: *ChipWhisperer/Native*

 4. Switch to the *Target Settings* tab. Set the following two options:
 
  a. Reader Hardware: *ChipWhisperer-USI*
  b. SmartCard Protocol: *SASEBO-W SmartCard OS*
   
 5. Press the *Master Connect* button, the scope and target should both show as connected:
 
    .. image:: /images/tutorials/basic/scard/allcon.png
 
 6. Under the *Scope Settings* tab, make the following changes:
  
  a. OpenADC-->Gain-->Setting: *35*
  b. OpenADC-->Trigger Setup-->Mode: *Rising Edge*
  c. CW Extra-->Clock Source: *TargetIO-IN*
  d. CW Extra-->Trigger Pins: Uncheck *Front Panel A*
  e. CW Extra-->Trigger Pins: Check *Target IO4 (Trigger Line)*
  f. CW Extra-->TargetIOn Pins-->TargetIO3: *USI-IN/OUT*
  g. OpenADC-->Clock Setup-->ADC Clock-->Source: *EXTCLK x4 via DCM*
  h. Press the *Reset ADC DCM* button in that area, confirm the *ADC Freq* reads 14.3 MHz indicating the clock routing is working.
  i. OpenADC-->Trigger Setup-->Total Samples: *5000*
  
 7. Finally press the *Capture 1* button. You should see a waveform like this:
 
    .. image:: /images/tutorials/basic/scard/waveform.png
   
 8. Currently the APDU is printed , see the 'Debug Logging' window. You will see output like this::
 
      APDU:  80  12  00  00  10  12  2b  7e  15  16  28  ae  d2  a6  ab  f7  15  88  09  cf  4f  3c  90  00  
      APDU:  80  04  04  00  10  04  2f  b7  e7  ce  f2  b8  09  92  0d  af  16  4a  81  30  3e  ef  9f  10  
      WARNING: USI parity error
      APDU:  00  c0  00  00  10  c0  34  2b  1f  28  5e  78  66  44  aa  f5  8e  eb  e6  cc  33  d7  90  00
      
    You can ignore the parity errors for now. You can also view the status in the Encryption Monitor to see input/output.
    
 9. You can now run a capture campaign and save the traces as before.
  