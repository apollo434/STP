***
The basic concept of STP:
1. The basic algorithm of STP
2. The actions of STP
3. How to update database STP

Q: STP only apply in L2?

A: So far, it is only L2 protocol.

**l2 topology**

![Alt text](/pic/L2_toplogy.png)

Q: What is Root Networking Bridge? and what is Specify Bridge?

A: Root Networking Bridge
   1. Root Networking Bridge is the only one that can create "BPDU" frame, other ones only transfer the "BPDU".
   2. When the topology changed, the Root Networking Bridge is the only one can make sure that all of the terminal or other Bridge know the actions.

A: Specify Bridge
   1. Which Bridge is the shortest one connected with Root Networking Bridge,which is the Specify Bridge. Look! the Specify Bridge describe the LAN scope.

   NOTE: the l2 topology could explain it.

Q: What's the status and character of port can be applied?

A: The status of port
  1. Disable
  2. Blocking
  3. Listening
  4. Learning
  5. Forwarding

  NOTE: Except "Disable" status, the others can deal with "BPDU" frame.

**Port finite state machine**

![Alt text](/pic/port_finite_status_machine.png)

A: The character of port
  1. Root
  2. Specify

  Note: the l2 topology could explain it


Q: What's the task of Root port and Specify port?

A: Root port send BPDU. Specify port recieve BPDU

Q: What's temporary loop?

A: Init the STP topology, it will show temporary loop, we use the port status changed to eliminate it.

Q: What's the algorithm the STP used?

A:

Networking Bridge ID & Port ID:

**Bridge & Port ID**
![Alt text](/pic/Bridge_ID_Port_ID.png)

The cost of port:
1. The lower cost, the higher priority!
2. The default cost based on the speed of port

STP Timer:
Every port and Networking Bridge are all applied Timer.

***
***
BPDU: which contain the informations that can determine which one will become Root Networking Bridge, and all of the status and character of other Networking Bridge.

**BPDU**
![Alt text](/pic/BPDU.png)

1. Configure BPDU
2. TCN(Topology Change Notification)

**BPDU version**

![Alt text](/pic/BPDU_vertion.png)

Q: What's time to configure BPDU?

A:

***

***
Regarding the old BPDU, please check the picture below:

![Alt text](/pic/BPDU_old.png)

Q: How to define the active topology?

A:
1. Select Root Bridge.
2. Select a Root port from all of its' in Root Bridge.
3. Scan all of the port, to allocate Specify Bridge and Specify port

All of the Three steps are named "Configuration Update"

The detailed and key informations of each step:

No.1 Select Root Bridge:

If the Bridge ID is not the best one, later, it will receive a configuration BPDU, which contain a better Root Bridge ID, then:

1. Accept this better BPDU, and mark down the key informations(Root Bridge ID and Timer)

2. Accordingly, Update the status and character of each port in this Bridge.

No.2 Select Root Port:

**NOTE**
1. Only Root Bridge do not has Root Port.
2. Only anti-Root Bridge do has only one Root Port.

![Alt text](/pic/select_root_port.png)

No.3 Select Specify Port:

**NOTE**
Each Bridge usually connected with at least one LAN, so, they must know the Specify Port with Each LAN.

![Alt text](/pic/select_specify_port.png)

Take a example as a summary:


![Alt text](/pic/example1.png)
**update cost topology**

![Alt text](/pic/example2.png)
**add a new Bridge 3**

![Alt text](/pic/example3.png)
**update topology**

**NOTE**
```
--------------------
  |
  | Root Bridge ID
  | Cost
  | Bridge ID
  | Port ID
  |
  V
```
***

***
Timer

![Alt text](/pic/bridge_timer.png)
**Bridge Timer**

![Alt text](/pic/port_timer.png)
**Port Timer**

**NOTE**

Each Bridge could save two copy Timer:
1. Administrator configuration
2. Root Bridge BPDU configuration

Look! Only Root Bridge use itself Timer configuration

![Alt text](/pic/fd_timer_flow.png)
**Forwarding Delay Timer Flow**

```
|---------|<------------------------------------------|
| Disable |<------------------------|                 |
|---------|<--------|               |                 |
    ^               |               |                 |
    |               |               |                 |
    |               |               |                 |
|---------|     |---------|      |--------|       |----------|
| Blocking|---->|Listening|----->|Learning|------>|Forwarding|
|---------|     |---------|      |--------|       |----------|
^^   ^^   ^^        ||               ||                ||
||   ||   ||        ||               ||                ||
||   ||   ===========                ||                ||
||   ||                              ||                ||
||    ================================                 ||
||                                                     ||
||                                                     ||
 =======================================================

-----> protocol determine
=====> administrator determine

```
**NOTE**
--------------------------------------------------------------------------
Listen ==> Forward Delay Timer ==> send BPDU
--------------------------------------------------------------------------
At this point, port can't be used to forward normal data frames. BPDU only
--------------------------------------------------------------------------

Why we need this status changed above, it is because of the deleting the temporary "LOOP"

***
***
Topology Change

Q: What's time the topology changed?

A:

1. anti-Forwarding <==> Forwarding Status Changed
2. Root Bridge ID Changed
3. Bridge port receive TCN

Shot Aging Timer: In order to inform the Forwarding Database need to be updated.

**NOTE**

If the topology has been changed, the changed information must be informed to all of the Bridge.

The progress:
1. Probe the Topology Change, and inform it to Root Bridge. (TCN BPDU)
2. Root Bridge inform all of the Bridge that the topology has been changed. (BPDU with TC flag)

When other Bridge receive the BDPU with TC flag, which will boot up the Short Aging Timer to update the Forwarding Database.

Take an example:

![Alt text](/pic/tcn_bpdu.png)
**TCN BPDU**

1. A2 <--X--> D1
2. A2 Change the Root Port, which connect with D2, the status of this port changed: Blocking ==> Forwarding
3. A2 boot up TCN Timer, then send TCN BPDU from new Root Port to D2
4. D2 receive the TCN BPDU, then respond TCA BPDU to A2, then boot up TCN Timer in D2, A2 receive the TCA BPDU, then stop TCN Timer.
5. D2 send TCN BPDU from Root Port to C1, C1 will do the same thing as step 4.
6. C1 receive the TCN BPDU, then boot up Topology Change Timer(35s), during 35s, C1 will send BPDU with TC flag.
7. When other Bridge receive the BPDU with TC flag, which will boot up Short Aging Timer. After 35s, the BPDU without TC flag, the Aging Timer boot up again, and stop Short Aging Timer.

***

***
BPDU Structure.

![Alt text](/pic/bpdu_str.png)



***
