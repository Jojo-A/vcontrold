# Linux Daemon for Viessmann Vito communication

vcontrold is a software daemon written in C for communication with the "Optolink" interface of Viessmann Vito heating controllers. Configuration is done via XML-files. The daemon offers an ASCII socket interface, that can be served with telnet or the [vclient](vclient.md) program.

The source code can be downloaded from [THIS](http://sourceforge.net/p/vcontrold/code/HEAD/tree/) SVN repository. Instructions for manual compilation can be found [HERE](Vcontrold-Kompilieren). There is a [feed](https://github.com/probonopd/vcontrold-for-openwrt) (makefile) on github as well as binaries for OpenWRT. Gentoo users can find an ebuild including init-script under [Bug #574964](https://bugs.gentoo.org/show_bug.cgi).

The configuration of the program is made in two XML-files:
- **vcontrold.xml**
  program-specific definitions
- **vito.xml**
  definitions of devices and commands
Default path for configuration files is /etc/vcontrold.

A "kill -1" reloads the configuration files. This can be used to reload commands and protocols, although this section won't be re-read (?).

**Usage:** <br />
vcontrold [-x xml-file] [-d <device>] [-l <logfile>] [-p port] [-s] [-n] [-i] [-g] <br />
-x path to XML configuration file (default is /etc/vcontrold) <br />
-d device, either *serial device* or *IP-address:port* for access via [ser2net](http://sourceforge.net/projects/ser2net) <br />
-l logfile <br />
-p TCP port on which the CLI interface is listening <br />
-s logging in syslog <br />
-n no fork, for testing purposes <br />
-i creates a file */temp/sim-.ini*, that will be used by the simulator vsim. Every command will be logged in the format *sent bytes = received bytes* <br />
-g debug mode

**CLI-commands:** <br />
***device:*** name and ID of the configured device <br />
***protocol:*** name of the protocol <br />
***commands:*** XML-file commands for the specific device <br />
***detail:*** detailed information about the command: execute command (?) <br />
***close:*** closes the communication channel (otherwise will stay open) <br />
***debug on|off:*** show/hide debug messages <br />
***unit on|off:*** switch scaling into predefined units on/off <br />
***reload:*** reload XML configuration files. this won't re-process the values under <unix>. This way, for commands that use *setaddr*, the HEX values can be added to the call.

```
vctrld>unit off
DEBUG:Sun Mar  2 14:44:32 2008 : Befehl: unit off
vctrld>settempWW 01 FF
DEBUG:Sun Mar  2 14:44:41 2008 : Befehl: settempWW 01 FF
DEBUG:Sun Mar  2 14:44:41 2008 : ClI Net: verbunden 192.168.1.112:3000 (FD:7)
DEBUG:Sun Mar  2 14:44:41 2008 : >SEND: 04
DEBUG:Sun Mar  2 14:44:41 2008 : Warte auf 05
DEBUG:Sun Mar  2 14:44:41 2008 : <RECV: 06
DEBUG:Sun Mar  2 14:44:44 2008 : <RECV: 05
DEBUG:Sun Mar  2 14:44:44 2008 : >SEND: 01
DEBUG:Sun Mar  2 14:44:44 2008 : >SEND: F4
DEBUG:Sun Mar  2 14:44:44 2008 : >SEND: 08
DEBUG:Sun Mar  2 14:44:44 2008 : >SEND: 04
DEBUG:Sun Mar  2 14:44:44 2008 : >SEND: 02
DEBUG:Sun Mar  2 14:44:44 2008 : >SEND: 01
DEBUG:Sun Mar  2 14:44:44 2008 : >SEND: FF
```

**Example session:**
```
$ telnet 192.168.1.2 1234
Trying 192.168.1.2...
Connected to 192.168.1.2.
Escape character is '^]'.
vctrld>commands
gettempA: Ermittle die Aussentemeratur in Grad C
gettempWW: Ermittle die Warmwassertemepratur in Grad C
vctrld>detail gettempWW
gettempWW: SEND 04;WAIT 05;SEND 01 F7 08 04 02;RECV 02 UT
        Unit: Temperatur
        Type: short
        Calc:  V/10
        Einheit: Grad Celsius
vctrld>gettempWW
54.299999 Grad Celsius
vctrld>unit off
vctrld>gettempWW
1F 02
vctrld>debug on
vctrld>gettempWW
DEBUG:Mon Feb 25 19:44:33 2008 : Befehl: gettempWW
DEBUG:Mon Feb 25 19:44:33 2008 : >SEND: 04
DEBUG:Mon Feb 25 19:44:33 2008 : Warte auf 05
DEBUG:Mon Feb 25 19:44:33 2008 : <RECV: 06
DEBUG:Mon Feb 25 19:44:36 2008 : <RECV: 05
DEBUG:Mon Feb 25 19:44:36 2008 : >SEND: 01
DEBUG:Mon Feb 25 19:44:36 2008 : >SEND: F7
DEBUG:Mon Feb 25 19:44:36 2008 : >SEND: 08
DEBUG:Mon Feb 25 19:44:36 2008 : >SEND: 04
DEBUG:Mon Feb 25 19:44:36 2008 : >SEND: 02
DEBUG:Mon Feb 25 19:44:36 2008 : <RECV: 1F
DEBUG:Mon Feb 25 19:44:36 2008 : <RECV: 02
1F 02
DEBUG:Mon Feb 25 19:44:36 2008 : Empfangen: 1F 02
vctrld>close
DEBUG:Mon Feb 25 19:44:52 2008 : Befehl: close
192.168.1.112:3000 geschlossen
vctrld>quit
DEBUG:Mon Feb 25 19:44:54 2008 : Befehl: quit
good bye!
DEBUG:Mon Feb 25 19:44:54 2008 : Verbindung beendet (fd:6)
Connection closed by foreign host.
```

For building and installation instructions see doc/INSTALL.md.

Please visit the OpenV Wiki for in-depth info and examples.
