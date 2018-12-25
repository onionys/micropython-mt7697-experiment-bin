# micropython-mt7697-experiment-bin

micropython mt7697 porting experiment version

    implementation version : 1.9.4
    platform : experiment

此版本為實作版本 bin 檔。以實驗目的釋出歡迎 bug 回報與測試。Source Code 預計 2019 年上半釋出。

適用 linkit7697 HDK 開發板。

MicroPython 官方網頁

    https://github.com/micropython/micropython

# 燒錄工具使用 MTK 官方提供的 uploader 

[https://github.com/MediaTek-Labs/mt76x7-uploader.git](https://github.com/MediaTek-Labs/mt76x7-uploader.git)


以 MacOSX 為例:

將 micropython_linkit7697_experiment_version.bin 複製到 uploader 資料夾中執行下列指令

	$ python ./upload.py -c /dev/tty.SLAB_USBtoUART -n ./da97.bin -t cm4 -f ./micropython_linkit7697_experiment_version.bin

# 已完成功能測試範例

## 內部檔案系統操作

### 重置檔案系統方式

斷電後，將 `Pin8` 接地後再上電。

該動作會將 linkit7697 內部檔案系統所存放的檔案全部刪除後寫入預設的 `main.py` 和 `boot.py`。

預設檔案目錄路徑為 `/flash`，容量約 400kb。

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
    # 按 USR BTN 測試

## PWM

    from machine import PWM
    # freq : 400 ~ 19000 Hz, 
    # duty : 0 ~ 1023
    p7 = PWM(7,freq=1024,duty=1000)
    p7.duty(200)

## ADC 

mt7697 ADC 功能限定支援 p14, p15, p16, p17 

所讀取的值為 0 (0.0V) ~ 4095 (2.45~2.55V) (以官方Datasheet為準)

    from machine import ADC
    p14 = ADC(14)
    val = p14.read()
    print(val)

## UART

mt7697 UART 功能限定 UART1，對應Pin 腳:

    p6 : UART1_TX
    p7 : UART1_RX

    # 115200 8N1
    from machine import UART
    uart = UART(1, 115200, 8, None, 1)

    # blocking read 
    data = uart.read()
    data = uart.read(12)

    # write 
    uart.write('hello')

nonblocking read: (timeout 單位為 ms)

    from machine import UART
    uart = UART(1, 115200, 8, None, 1, timeout = 1000)
    data = uart.read()

## Timer

period 單位為 ms

    def hello(x):
        print('hello')
	
	from machine import Timer
	t0 = Timer(0)
	t0.init(mode=Timer.PERIODIC, callback=hello, period=1000)

	# stop timer
	t0.deinit()

# SPI Master (default micropython Soft SPI)

目前只提供 Master mode，使用 Micropython 官方版本軟體 SPI。

    # SPI CS   : p10
	# SPI MOSI : p11
    # SPI MISO : p12
    # SPI SCK  : p13

    from machine import SPI, Pin
	cs = Pin(10, mode=Pin.OUT)
	cs(0)
	spi = SPI(mosi = 11, miso = 12, sck = 13)
	spi.write(bytes([1,2,3,4,5]))

[ref](https://github.com/micropython/micropython/wiki/Hardware-API)

# I2C Master 

目前只提供 Master mode，使用 Micropython 官方版本軟體 I2C。

    # I2C SCL : p8
    # I2C SDA : p9

    from machine import I2C
	from machine import Pin
	i2c = I2C(scl=Pin(8), sda=Pin(9))
	i2c.scan()

[ref](https://github.com/micropython/micropython/wiki/Hardware-API)

## WDT

Watch Dog timeout 限定提供一個

timeout : 0 ~ 30 秒

    from machine import WDT
	wdt = WDT(id=0, timeout=5)
	# wait 5 second for reboot
	wdt.feed()

# WiFi STA mode TCP client connection

    import network, usocket
	wlan = network.WLAN()
	wlan.connect('your_ap_ssid', 'password')
	soc = usocket.socket(usocket.AF_INET, usocket.SOCK_STREAM)
	port = 9999
	addr = ('remote_server_IP', port)
	soc.connect(addr)
	soc.send(b'hello world')
	print(soc.recv(1024))

# 其他功能

實作中

# 開發與維護人員

onionys

    mail: onionys@gmail.com
    github: https://github.com/onionys

tomlin-ntust

    mail: tomlin.ntust@gmail.com
    github: https://github.com/tomlin-ntust/


