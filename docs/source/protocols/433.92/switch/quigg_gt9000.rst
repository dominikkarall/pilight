.. |yes| image:: ../../../images/yes.png
.. |no| image:: ../../../images/no.png

.. role:: underline
   :class: underline

Globaltronics Quigg GT-9000
===========================

+------------------+-------------+
| **Feature**      | **Support** |
+------------------+-------------+
| Sending          | |yes|       |
+------------------+-------------+
| Receiving        | |yes|       |
+------------------+-------------+
| Config           | |yes|       |
+------------------+-------------+

.. rubric:: Supported Brands

+-------------------------------------+---------------+
| **Brand**                           | **Protocol**  |
+-------------------------------------+---------------+
| Globaltronics GT-FSI-09 / GT-9000   | quigg_gt9000  |
+-------------------------------------+---------------+

.. rubric:: Sender Arguments

.. code-block:: console
   :linenos:

   -t --on           send an on signal to a device
   -f --off          send an off signal to a device
   -i --id=id        control the device with this system id (1 ... FIXME)
   -u --unit=unit    control the device with this unit code (0 ... 15)

.. rubric:: Config

.. code-block:: json
   :linenos:

   {
     "devices": {
       "dimmer": {
         "protocol": [ "quigg_gt9000" ],
         "id": [{
           "id": FIXME,
           "unit": 0
         }],
         "state": "off"
       }
     },
     "gui": {
       "Lamp": {
         "name": "TV Backlit",
         "group": [ "Living" ],
         "media": [ "all" ]
       }
     }
   }

+------------------+-----------------+
| **Option**       | **Value**       |
+------------------+-----------------+
| id               | 1 - FIXME       |
+------------------+-----------------+
| unit             | 0 - 15          |
+------------------+-----------------+
| state            | on / off        |
+------------------+-----------------+

.. rubric:: Protocol

The quigg_switch protocol sends 50 pulses like this

.. code-block:: console

   416 1040 1040 624 1040 624 1040 624 416 1040 416 1040 416 1040 1040 624 416 1040 416 1040 1040 624 416 1040 416 1040 1040 624 1040 624 1040 624 416 1040 1040 624 1040 624 416 1040 1040 624 1040 624 416 1040 416 1040 2912 7072

The the last two pulses are the ``footer`` (3000, 7000). These are meant to identify the pulses as genuine. We don't use them for further processing. The next step is to transform this output into 24 groups of 2 pulses (and thereby dropping the ``footer`` pulses).

.. code-block:: console

   416 1040
   1040 624
   1040 624
   1040 624
   416 1040
   416 1040
   416 1040
   1040 624
   416 1040
   416 1040
   1040 624
   416 1040
   416 1040
   1040 624
   1040 624
   1040 624
   416 1040
   1040 624
   1040 624
   416 1040
   1040 624
   1040 624
   416 1040
   416 1040

If we now analyse these groups we can distinguish two types of groups:

#. ``416 1040``
#. ``1040 624``

So the first group is defined by a short 1st and 2nd long and the second group by a long 1st and 2nd short pulse. The first one defines a 0 and the second pair defines 1. We then get the following output:

.. code-block:: console

	 011100010010011101101100

We can group the sequence of bits into the following groups A to D.
Each of the groups of bits (A to D) has a specific meaning:

+-----------+-----------+----------------------+
| **Group** | **Bit #** | **Description**      |
+-----------+-----------+----------------------+
| A         | 0 to 3    | 1st part systemcode  |
+-----------+-----------+----------------------+
| B         | 3 to 19   | encrypted systemcode |
+-----------+-----------+----------------------+
| C         | 16 to 19  | on/off statecode     |
+-----------+-----------+----------------------+
| D         | 20 to 24  | unit                 |
+-----------+-----------+----------------------+

CONTINUE HERE
So this code represents:

.. code-block:: console

  "id": 2816,
  "unit": 1
  "state": Off

.. rubric:: Examples

CLI command:

.. code-block:: console

   pilight-send -p quigg_gt7000 -i 2816 -u 1 -f

.. rubric:: Comment

Extracting the system code id from an existing Globaltronics GT-7000 remote control device either requires a special version of the BPF, or you need to bypass the BPF.

After insertion of batteries the GT-7000 defaults to system code id #2816. Pressing the button "Neuer Code" located in the battery compartment, will trigger the generation of a new system code id. These are generated in sequential order, for the current quigg_switch protocol driver the id's are:

.. code-block:: console

   2816, 1792, 3840, 128, 2176, 1152, 3200, 640, 2688, 1664, 3712, 384, 2432, 1408, 3456, 896, 2944, 1920, 3968, ....

To let the device learn a new value, press the learning mode button on the switch and send the appropriate CLI command with pilight-send (configure a switch to be unit #2 and system code id #29):

.. code-block:: console

   pilight-send -p quigg_gt7000 -i 29 -u 2 -l -t

The device learns that it has now system code id #29 and that it is unit #2 and enters ON mode (e.q. the switch is turned on). If the switch is not connected to power for an extended period of time, it will loose its configuration and reset to the default id #2816 unit #0. QUIGG_GT7000 compatible switches with integrated dimmer require that you configure the quigg_screen protocol in addition to the quigg_gt7000 protocol. On the webgui you will get a separate button to dimm the device up and down.

Subsequently the switch unit #1 with system code id #2816 is turned off.
