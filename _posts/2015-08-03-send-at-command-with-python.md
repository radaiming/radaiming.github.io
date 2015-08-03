---
layout: post
title: "send AT command with Python"
categories: linux
---

Many 3G modems expose one or several serial ports to linux system, some of them could response to AT commands, you could use picocom to test, like:

~~~~~~~~
# picocom /dev/ttyUSB0
picocom v1.7

port is        : /dev/ttyUSB0
flowcontrol    : none
baudrate is    : 9600
parity is      : none
databits are   : 8
escape is      : C-a
local echo is  : no
noinit is      : no
noreset is     : no
nolock is      : no
send_cmd is    : sz -vv
receive_cmd is : rz -vv
imap is        :
omap is        :
emap is        : crcrlf,delbs,

Terminal ready
ATI
Manufacturer: huawei
Model: E3131
Revision: 21.158.13.03.112
IMEI: I'd like to hide this :)
+GCAP: +CGSM,+DS,+ES

OK
AT+CSQ
+CSQ: 19,99

OK
~~~~~~~~

With [pyserial](http://pyserial.sourceforge.net/), we could send AT commands with Python, a simple example is:

{% highlight python %}
#!/usr/bin/env python2
# -*- coding: utf-8 -*-

import serial
import sys

def main():
    tty_dev = raw_input('tell me the serial port path: ')
    try:
        ser = serial.Serial(tty_dev, 9600, timeout=0.1)
        print 'now you could start sending AT command:'
        while True:
            at_cmd = raw_input('> ') + '\r\n'
            ser.write(at_cmd)
            sys.stdout.write(ser.read(1024))
    except (serial.SerialException, OSError, EOFError):
        pass
    finally:
        if 'ser' in locals() and isinstance(ser, serial.serialposix.Serial):
            ser.close()

if __name__ == '__main__':
    main()
{% endhighlight %}

now run it:

~~~~~~~~
# ./send_at_cmd.py
tell me the serial port path: /dev/ttyUSB0
now you could start sending AT command:
> AT
AT
OK
> ATI
ATI
Manufacturer: huawei
Model: E3131
Revision: 21.158.13.03.112
IMEI: I'd like to hide this :)
+GCAP: +CGSM,+DS,+ES

OK
> AT+CSQ
AT+CSQ
+CSQ: 24,99

OK
>
~~~~~~~~
