# micropython-mt7697-experiment-bin

The official micropython porting of the linkit7697 HDK board for experiment and testing, and it is welcome to bugs report. 

    implementation version : 1.9.4
    platform : experiment

The Source Code Project is under preparing for release in the first half of 2019.


# mu editor (preparing)

Now we are preparing the mu editor for mt7697 micropython and going to release test version ASAP.


# update log

date : 2019-02-02

- transfer micropython version to 1.10

date : 2019-01-28

- add help() function to show brief info of linkit7697 board.
- add machine.UARTdefault set `9600 8 N 1`
- add RTC function
- fix PWM duty cycle bug


# upload tool 

Upload the bin file by using the uploader downloaded from:

   [MediaTek-Labs/mt76x7-uploader.git](https://github.com/MediaTek-Labs/mt76x7-uploader.git)

for example in MacOSX:

Copy bin file `micropython_linkit7697_experiment_version.bin` to the directory of the `uploader` and execute the following command.

	$ python ./upload.py -c /dev/tty.SLAB_USBtoUART -n ./da97.bin -t cm4 -f ./micropython_linkit7697_experiment_version.bin


# Connect to MicroPython REPL via USB Serial Port

Plug linkit7697 HDK to your computer via USB, then use serial connection application such as putty or minicom to connect with 
MicroPython REPL. The serial port configuration parameters are :

    115200 8 N 1

The Serial Port window of Arduino IDE is an easy way to establish connection with linkit7697 MicroPython REPL.


# Function Test Examples

## Internal File System Operation

### Reset the internal file system 

Attention! this action will delete all file in the internal file system and reset `main.py` and `boot.py` by default content.

1. shutdown the linkit 7697 board, then connect Pin `p8` to GND。

2. power on linkit 7697 board.

the default is `/flash` with capacity about 400kb。

    # list all file
    import uos
	print(uos.listdir())

	# file write
	f = open('test.txt','w')
	f.write('hello world\n')
	f.close()

    # file read
	f = open('test.txt')
	for line in f:
	    print(line)


## Help()

show brief info of linkit7697 board:

    help()

print


    Welcome to MicroPython on the linkit7697 HDK!
    
    For generic online docs please visit http://docs.micropython.org/
    
    For release info please visit https://github.com/onionys/micropython-mt7697-experiment-bin
    
     linkit 7697 HDK Pin map
                     /--------------------\
                     |* GND          3V3 *|
           UART0_RX--|* P0           P6  *|--UART1_TX-EINT20-(USR-BTN)
     EINT2-UART0_TX--|* P1           P7  *|--UART1_RX--------(USR-LED)
     EINT0-----------|* P2           P8  *|--HW_I2C1_CLK
     EINT22----------|* P3           P9  *|--HW_I2C1_DATA
              IR_RX--|* P4           P10 *|--HW_SPI_CS0
              IR_TX--|* P5           P11 *|--HW_SPI_MOSI
                     |* GND          P12 *|--HW_SPI_MISO
          (RST-BTN)--|* RST          P13 *|--HW_SPI_SCK
                     |* 3V3          P14 *|--ADC_IN0
                     |* 5V           P15 *|--ADC_IN1
                     |* D-           P16 *|--ADC_IN2
                     |* D+           P17 *|--ADC_IN3
                     |* GND          GND *|
                     | [RST-BTN] [USR-BTN]|
                     |       /-----\      |
                     |       |     |      |
                     ----------------------
                               USB
     PWM: P0~P17
    
    Control commands:
      CTRL-A        -- on a blank line, enter raw REPL mode
      CTRL-B        -- on a blank line, enter normal REPL mode
      CTRL-C        -- interrupt a running program
      CTRL-D        -- on a blank line, do a soft reset of the board
      CTRL-E        -- on a blank line, enter paste mode
    
    For further help on a specific object, type help(obj)
    For a list of available modules, type help('modules')


## Pin

output

    from machine import Pin
    p7 = Pin(7, Pin.OUT)

    # set output high
    p7(1)

    # set output low
    p7(0)

input

    from machine import Pin
    p6 = Pin(6, Pin.IN)
    # return Pin7 input level 1: high, 0:low
    lv = p6()
    print(lv)

irq

    from machine import Pin
    p6 = Pin(6)
    def hello(x):
        print('hello')
    p6.irq(hello, Pin.IRQ_RISING)
    # use USR BTN for test

## PWM

    from machine import PWM
    # freq : 400 ~ 19000 Hz, 
    # duty : 0 ~ 1024
    p7 = PWM(7,freq=1024,duty=1000)
    p7.duty(200)

## ADC 

ADC is supported by following pins : p14, p15, p16, p17.

The value returned is integer in the range of 0 (0.0V) ~ 4095 (2.45~2.55V).

    from machine import ADC
    p14 = ADC(14)
    val = p14.read()
    print(val)

## UART

UART is supported by UART1 with following pins:

    # p6 : UART1_TX
	# p7 : UART1_RX

    UART(1, baudrate=9600, bits=8, parity=None, stop=1, timeout=-1)
    
    only support UART 1
    
    baudrate :
        110, 300, 1200, 2400, 4800, 9600(defualt), 19200, 38400, 57600,
        115200, 230400, 460800, 921600
    
    bits :
        5, 6, 7, 8 (defualt)
    
    parity :
        None (or 0)
        1 (ODD:default) 
        2 (EVEN)
    
    stop :
        1(defualt), 2
    
example:

    from machine import UART
    uart = UART(1,115200)
    uart = UART(1,115200,8,None,1)

    # blocking read 
    data = uart.read()
    data = uart.read(12)

    # write 
    uart.write('hello')

nonblocking read: (timeout unit : ms)

    from machine import UART
    uart = UART(1, 115200, 8, None, 1, timeout = 1000)
    data = uart.read()

## Timer

    def hello(x):
        print('hello')
	
	from machine import Timer
	t0 = Timer(0)

	# period unit : ms
	t0.init(mode=Timer.PERIODIC, callback=hello, period=1000)

	# stop timer
	t0.deinit()

## SPI Master (default micropython Soft SPI)

Master mode only, implemented by MicroPython soft SPI.

    # SPI CS   : p10
	# SPI MOSI : p11
    # SPI MISO : p12
    # SPI SCK  : p13

    from machine import SPI, Pin
	cs = Pin(10, mode=Pin.OUT)
	cs(0)
	spi = SPI(mosi = 11, miso = 12, sck = 13)
	spi.write(bytes([1,2,3,4,5]))

## RTC 

    from machine import RTC
    my_rtc = RTC()

    # init
    my_rtc.init() # enable RTC

    # set new datetime:
    new_datetime = (2019,1,28,14,22,30) tuple : (year, mon, day, hour, min, sec)
    my_rtc.datetime(new_datetime)   # The RTC is enabled
                                    # if had not be enabled yet.

    # get datetime
    now = my_rtc.datetime() # return tuple of datetime
                            # (year, mon, day, hour, min, sec)
                            # ex: (2019,1,28,14,22,35)

[ref](https://github.com/micropython/micropython/wiki/Hardware-API)

## I2C Master 

Master mode only, implemented by MicroPython soft I2C.

    # I2C SCL : p8
    # I2C SDA : p9

    from machine import I2C
	from machine import Pin
	i2c = I2C(scl=Pin(8), sda=Pin(9))
	i2c.scan()

[ref](https://github.com/micropython/micropython/wiki/Hardware-API)

## WDT

Only support one Watch Dog.

timeout : 0 ~ 30 秒

    from machine import WDT
	wdt = WDT(id=0, timeout=5)
	# wait 5 second for reboot
	wdt.feed()

## WiFi STA mode TCP client connection

    import network, usocket
	wlan = network.WLAN()
	wlan.connect('your_ap_ssid', 'password')
	soc = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM)
	port = 9999
	addr = ('remote_server_IP', port)
	soc.connect(addr)
	soc.send(b'hello world')
	print(soc.recv(1024))

# Other functions

Still in working

# Developers

onionys

    mail: onionys@gmail.com
    github: https://github.com/onionys

tomlin-ntust

    mail: tomlin.ntust@gmail.com
    github: https://github.com/tomlin-ntust/


