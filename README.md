todbotさんによるCircuitPythonのトリック集です。大変有用な内容なのでリポジトリをforkさせていただき日本語化しました。

元のリポジトリはこちら
https://github.com/todbot/circuitpython-tricks

# CircuitPythonのトリック集

todbotさんがCircuitPythonのプログラミングの中で見つけられたコツとトリックを集めたものです。

## 目次
* [入力](#入力)
   * [ボタンのデジタル入力を読み取る](#ボタンのデジタル入力を読み取る)
   * [ポテンショメータを読み取る](#ポテンショメータを読み取る)
   * [静電容量タッチピンを読み取る](#静電容量タッチピンを読み取る)
   * [ロータリーエンコーダを読み取る](#ロータリーエンコーダを読み取る)
   * [ピンやボタンのデバウンス](#ピンやボタンのデバウンス)
   * [複数のピンをリスト化してデバウンス](#複数のピンをリスト化してデバウンス)
* [出力](#出力)
   * [ピンにHIGH/LOWを出力 (例: LEDのオン・オフ)](#ピンにHIGHまたはLOWを出力)
   * [DACピンにアナログ値を出力](#DACピンにアナログ値を出力)
   * [PWMピンにアナログ値を出力](#PWMピンにアナログ値を出力)
   * [Neopixelを制御](#Neopixelを制御)
* [Neopixel(WS2812B) / Dotstar(APA102)](#NeopixelとDotstar)
   * [マイコンに搭載されているNeoPixelを虹色に変化させる](#マイコンに搭載されているNeoPixelを虹色に変化させる)
   * [LEDテープに虹のグラデーションを表示](#LEDテープに虹のグラデーションを表示)
   * [LEDテープに流れ星のエフェクトを表示](#LEDテープに流れ星のエフェクトを表示)
* [USB](#usb)
   * [USBが接続されているかを検出](#USBが接続されているかを検出)
   * [CIRCUITPYのディスクサイズと空き容量を取得](#CIRCUITPYのディスクサイズと空き容量を取得)
   * [コードからUF2 bootloaderをリセット](#コードからUF2-bootloaderをリセット)
* [USBシリアル](#usbシリアル)
   * [USBシリアルに表示](#USBシリアルに表示)
   * [USBシリアルから入力を受け付ける（ブロッキング）](#USBシリアルから入力を受け付けるブロッキング)
   * [USBシリアルから入力を受け付ける（ほぼノンブロッキング）](#USBシリアルから入力を受け付けるほぼノンブロッキング)
   * [USBシリアルからキーを読み取る](#USBシリアルからキーを読み取る)
* [計算タスク](#計算タスク)
   * [テキストをフォーマットする](#テキストをフォーマットする)
   * [f文字列でテキストをフォーマットする](#f文字列でテキストをフォーマットする)
   * [configファイルを利用する](#configファイルを利用する)
* [より複雑なタスク](#より複雑なタスク)
   * [値のマッピング](#値のマッピング)
   * [時間の計測](#時間の計測)
   * [Ctrl-Cを押してもコードが停止しないようにする](#preventing-ctrl-c-from-stopping-the-program)
   * [Raspberry Pi Picoをセーフモードで起動できるようにする](#Raspberry-Pi-Picoをセーフモードで起動できるようにする)
* [ネットワーク](#ネットワーク)
   * [WiFiをスキャンして信号強度順にSSIDを表示 (ESP32-S2)](#WiFiをスキャンして信号強度順にSSIDを表示-ESP32-S2)
   * [IPアドレスにpingを送信 (ESP32-S2)](#IPアドレスにpingを送信-esp32-s2)
   * [JSONファイルを取得 (ESP32-S2)](#JSONファイルを取得-esp32-s2)
   * [secrets.pyってなに？](#secretspyってなに？)
* [I2C](#i2c)
   * [I2Cバスをスキャンする](#I2Cバスをスキャンする)
* [ボードの情報](#ボードの情報)
   * [空きメモリの容量を表示する](#空きメモリの容量を表示する)
   * [microcontroller.pinとboardのマッピング状況を表示](#microcontrollerpinとboardのマッピング状況を表示)
   * [ボード名の表示](#ボード名の表示)
   * [ひとつのcode.pyで複数のボードに対応する](#ひとつのcodepyで複数のボードに対応する)
* [Hacks](#hacks)
   * [REPLを使う](#REPLを使う)
      * [コアモジュールとライブラリの一覧を表示](#コアモジュールとライブラリの一覧を表示)
      * [ワンライナー](#ワンライナー)
* [Python情報](#python情報)
   * [コアモジュール以外のどんなライブラリがimportされているかを表示](#コアモジュール以外のどんなライブラリがimportされているかを表示)
   * [グローバル変数の一覧を表示](#グローバル変数の一覧を表示)
* [ホスト側のタスク](#ホスト側のタスク)
   * [CircuitPythonのライブラリをインストールする](#CircuitPythonのライブラリをインストールする)
      * [circupでライブラリをインストールする](#circupでライブラリをインストールする)
      * [cpコマンドでライブラリをコピーする](#cpコマンドでライブラリをコピーする)

## 入力

### ボタンのデジタル入力を読み取る

  ```python
  import board
  from digitalio import DigitalInOut, Pull
  button = DigitalInOut(board.D3) # デフォルトでは入力
  button.pull = Pull.UP # 内部のプルアップ抵抗を有効化する
  print(button.value)  # False == ボタンが押された
  ```

### ポテンショメータを読み取る 
  ```py
  import board
  import analogio
  potknob = analogio.AnalogIn(board.A1)
  position = potknob.value  # 0-65535の範囲の値
  pos = potknob.value // 256  # 0-255の範囲にする
  ```

### 静電容量タッチピンを読み取る
  ```py
  import touchio
  import board
  touch_pin = touchio.TouchIn(board.GP6)
  # Raspberry Pi Pico / RP2040の場合、1MΩのプルダウン抵抗が各入力に必要
  if touch_pin.value: 
    print("touched!")
  ```

### ロータリーエンコーダを読み取る
  ```py
  import board
  import rotaryio
  encoder = rotaryio.IncrementalEncoder(board.GP0, board.GP1) # Picoの場合、GP0、GP1のように連続したピンである必要がある
  print(encoder.position)  # ゼロからスタートして、マイナスまたはプラスの値が表示される
  ```

### ピンやボタンのデバウンス 
  ```py
  import board
  from digitalio import DigitalInOut, Pull
  from adafruit_debouncer import Debouncer
  button_in = DigitalInOut(board.D3) # defaults to input
  button_in.pull = Pull.UP # turn on internal pull-up resistor
  button = Debouncer(button_in)
  while True:
    button.update()
    if button.fell:
      print("press!")
    if button.rose:
      print("release!")
  ```

### 複数のピンをリスト化してデバウンス
  ```py
  import board
  from digitalio import DigitalInOut, Pull
  from adafruit_debouncer import Debouncer
  pins = (board.GP0, board.GP1, board.GP2, board.GP3, board.GP4)
  buttons = []   # デバウンスしたいオブジェクトのリストが格納される
  for pin in pins:
    tmp_pin = DigitalInOut(pin) # defaults to input
    tmp_pin.pull = Pull.UP # 内部のプルアップ抵抗を有効化する
    buttons.append( Debouncer(tmp_pin) )
  while True:
    for i in range(len(buttons)):
      buttons[i].update()
      if buttons[i].fell:
        print("button",i,"pressed!")
      if buttons[i].rose:
        print("button",i,"released!")
  ```
        
## 出力

### ピンにHIGHまたはLOWを出力
  ```py
  import board
  import digitalio
  ledpin = digitalio.DigitalInOut(board.D2)
  ledpin.direction = digitalio.Direction.OUTPUT
  ledpin.value = True
  ```

### DACピンにアナログ値を出力
ボードによってDACピンが異なります。
  ```py
  import board
  import analogio
  dac = analogio.AnalogOut(board.A0)  # Trinket M0とQT Pyの場合
  dac.value = 32768   # 0-65535の範囲の中央の値
  ```

### PWMピンにアナログ値を出力
  ```py
  import board
  import pwmio
  out1 = pwmio.PWMOut(board.MOSI, frequency=25000, duty_cycle=0)
  out1.duty_cycle = 32768  # 0-65535の範囲の中央の値 = 50 % デューティサイクル
  ```

### Neopixelを制御
  ```py
  import neopixel
  led = neopixel.NeoPixel(board.NEOPIXEL, 1, brightness=0.2)
  led[0] = 0xff00ff
  led[0] = (255,0,255)  # 0xff00ffを指定したのと同じ

  ```

## NeopixelとDotstar

### マイコンに搭載されているNeoPixelを虹色に変化させる

`_pixelbuf` または `adafruit_pypixelbuf` の一部である組み込みの `colorwheel()` 関数を使用します。
この関数は、0～255の色相を指定して、`(R,G,B)`のタプルを返します。以下にその使い方を示します。
これは、`neopixel`の代わりに`adafruit_dotstar`でも動作します。

```py
import time
import board
import neopixel
led = neopixel.NeoPixel(board.NEOPIXEL, 1, brightness=0.4)
while True:
  led.fill( neopixel._pixelbuf.colorwheel((time.monotonic()*50)%255) )
  time.sleep(0.05)
```

### LEDテープに虹のグラデーションを表示

デモは [こちら](https://twitter.com/todbot/status/1397992493833097218).

```py
import time, random
import board, neopixel
num_leds = 16
leds = neopixel.NeoPixel(board.D2, num_leds, brightness=0.4, auto_write=False )
delta_hue = 256//num_leds
speed = 10  # 数字が大きい = 色の変化が速い
i=0
while True:
  for l in range(len(leds)):
    leds[l] = neopixel._pixelbuf.colorwheel( int(i*speed + l * delta_hue) % 255  )
  leds.show()  # 全部のLEDの値を変更したあとに更新をかける
  i = (i+1) % 255
  time.sleep(0.05)
```

### LEDテープに流れ星のエフェクトを表示
```py
import time, random
import board, neopixel
num_leds = 16
leds = neopixel.NeoPixel(board.D2, num_leds, brightness=0.4, auto_write=False )
my_color = (55,200,230)
dim_by = 20  # 数字が大きい = 流れ星の尾が短い
pos = 0
while True:
  leds[pos] = my_color
  leds[0:] = [[max(i-dim_by,0) for i in l] for l in leds] # dim all by (dim_by,dim_by,dim_by)
  pos = (pos+1) % num_leds  # 次の位置に移動する
  leds.show()  # 全部のLEDの値を変更したあとに更新をかける
  time.sleep(0.05)
```


## USB

### USBが接続されているかを検出
  ```py
  def is_usb_connected():
    import storage
    try:
      storage.remount('/', readonly=False)  # attempt to mount readwrite
      storage.remount('/', readonly=True)  # attempt to mount readonly
    except RuntimeError as e:
      return True
    return False
  is_usb = "USB" if is_usb_connected() else "NO USB"
  print("USB:", is_usb)
  ```

### CIRCUITPYのディスクサイズと空き容量を取得
  ```py
  import os
  fs_stat = os.statvfs('/')
  print("Disk size in MB", fs_stat[0] * fs_stat[2] / 1024 / 1024)
  print("Free space in MB", fs_stat[0] * fs_stat[3] / 1024 / 1024)
  ```

### コードからUF2 bootloaderをリセット 
  ```py
  import micrcocontroller
  microcontroller.on_next_reset(microcontroller.RunMode.BOOTLOADER)
  microcontroller.reset()
  ```

## USBシリアル

### USBシリアルに表示
  ```py
  print("hello there")  # prints a newline
  print("waiting...", end='')   # end=''とすることで改行しなくなる
  ```

### USBシリアルから入力を受け付ける（ブロッキング）
  ```py
  while True:
    print("Type something: ", end='')
    my_str = input()  # キー入力をしてENTERを押す
    print("You entered: ", my_str)
  ```

### USBシリアルから入力を受け付ける（ほぼノンブロッキング）
  ```py
  import time
  import supervisor
  print("Type something when you're ready")
  last_time = time.monotonic()
  while True:
    if supervisor.runtime.serial_bytes_available:
      my_str = input()
      print("You entered:", my_str)
    if time.monotonic() - last_time > 1:  # 1秒毎に表示
      last_time = time.monotonic()
      print(int(last_time),"waiting...")
  ```

### USBシリアルからキーを読み取る
  ```py
  [tbd]

  ```


## 計算タスク

### テキストをフォーマットする
  ```py
  name = "John"
  fav_color = 0x003366
  body_temp = 98.65
  print("name:%s color:%06x thermometer:%2.1f" % (name,fav_color,body_temp))
  'name:John color:ff3366 thermometer:98.6'
  ```

### f文字列でテキストをフォーマットする
（QTPy M0のような小さなCircuitPythonでは機能しません）

```py
  name = "John"
  fav_color = 0x003366
  body_temp = 98.65
  print(f"name:{name} color:{color:06x} thermometer:{body_temp:2.1f}")
  'name:John color:ff3366 thermometer:98.6'
```


### configファイルを利用する
  ```py
  # my_config.py
  config = {
    "username": "Grogu Djarin",
    "password": "ig88rules",
    "secret_key": "3a3d9bfaf05835df69713c470427fe35"
  }
  # code.py
  from my_config import config
  print("secret:", config['secret_key'])
  'secret: 3a3d9bfaf05835df69713c470427fe35'
  ```


## より複雑なタスク

### 値のマッピング
  ```py
  # Arduino map()のようなシンプルなマッピング
  def map_range(s, a, b):
      (a1, a2), (b1, b2) = a, b
      return  b1 + ((s - a1) * (b2 - b1) / (a2 - a1))
  # 0-0123の範囲の値を0.0-1.0の範囲にマッピングする
  out = map_range( in, (0,1023), (0.0,1.0) )
  ```

### 時間の計測
  ```py
  import time
  start_time = time.monotonic() # 電源投入時
  do_something()
  elapsed_time = time.monotonic() - start_time
  print("do_something took %f seconds" % elapsed_time)
  ```

### Ctrl-Cを押してもコードが停止しないようにする

メインループ内に `try`/`except KeyboardInterrupt` を記述してCtrl-Cを押したことを検出します。

```py
while True:
  try:
    print("Doing something important...")
    time.sleep(0.1)
  except KeyboardInterrupt:
    print("Nice try, human! Not quitting.")
```

Ctrl-Cを押して、優雅に(LEDを消して、終了メッセージを表示してから)シャットダウンすることもできます。

```py
import time, random
import board, neopixel, adafruit_pypixelbuf
leds = neopixel.NeoPixel(board.NEOPIXEL, 1, brightness=0.4 )
while True:
  try:
    rgb = adafruit_pypixelbuf.colorwheel(int(time.monotonic()*75) % 255)
    leds.fill(rgb) 
    time.sleep(0.05)
  except KeyboardInterrupt:
    print("shutting down nicely...")
    leds.fill(0)
    break  # gets us out of the while True
```


### Raspberry Pi Picoをセーフモードで起動できるようにする

他のRP2040搭載ボード（QTPy RP2040等）でも機能します。

  ```py
  # このコードをboot.pyとしてRaspberry Pi PicoのCIRCUITPY driveに保存しておく
  # from https://gist.github.com/Neradoc/8056725be1c209475fd09ffc37c9fad4
  # Picoがロックアップしてしまったときに便利
  import board
  import time
  from digitalio import DigitalInOut,Pull
  import time
  led = DigitalInOut(board.LED)
  led.switch_to_output()

  safe = DigitalInOut(board.GP14)
  safe.switch_to_input(Pull.UP)

  def reset_on_pin():
	if safe.value is False:
	import microcontroller
	microcontroller.on_next_reset(microcontroller.RunMode.SAFE_MODE)
	microcontroller.reset()

    led.value = False
    for x in range(16):
	reset_on_pin()
	led.value = not led.value
	time.sleep(0.1)
    
  ```

## ネットワーク

### WiFiをスキャンして信号強度順にSSIDを表示 (ESP32-S2)

```py
networks = []
for network in wifi.radio.start_scanning_networks():
  networks.append(network)
wifi.radio.stop_scanning_networks()
networks = sorted(networks, key=lambda net: net.rssi, reverse=True)
for network in networks:
  print("ssid:",network.ssid, "rssi:",network.rssi)
```

### IPアドレスにpingを送信 (ESP32-S2)
```py
import time
import wifi
import ipaddress
from secrets import secrets
ip_to_ping = "1.1.1.1"
wifi.radio.connect(ssid=secrets['ssid'],password=secrets['password'])
print("my IP addr:", wifi.radio.ipv4_address)
print("pinging ",ip_to_ping)
ip1 = ipaddress.ip_address(ip_to_ping)
while True:
    print("ping:", wifi.radio.ping(ip1))
    time.sleep(1)
```

### JSONファイルを取得 (ESP32-S2)
```py
import time
import wifi
import socketpool
import ssl
import adafruit_requests
from secrets import secrets
wifi.radio.connect(ssid=secrets['ssid'],password=secrets['password'])
print("my IP addr:", wifi.radio.ipv4_address)
pool = socketpool.SocketPool(wifi.radio)
session = adafruit_requests.Session(pool, ssl.create_default_context())
while True:
    response = session.get("https://todbot.com/tst/randcolor.php")
    data = response.json()
    print("data:",data)
    time.sleep(5)
```

### `secrets.py`ってなに？
AdafruitのWiFiライブラリで使用するファイルです。
基本的なWiFi接続を行うような場合に使います。
```py
# secrets.py
secrets = {
  "ssid": "Pretty Fly for a WiFi",
  "password": "donthackme123"
}
# code.py
from secrets import secrets
print("your WiFi password is:", secrets['password'])
```

## I2C

### I2Cバスをスキャンする
参照: https://learn.adafruit.com/circuitpython-essentials/circuitpython-i2c#find-your-sensor-2985153-11

```py
import board
i2c = board.I2C() # or busio.I2C(pin_scl,pin_sda)
while not i2c.try_lock():  pass
print("I2C addresses found:", [hex(device_address)
    for device_address in i2c.scan()])
i2c.unlock()
```


## ボードの情報

### 空きメモリの容量を表示する

from: https://learn.adafruit.com/welcome-to-circuitpython/frequently-asked-questions
```py
import gc
print( gc.mem_free() )
```

### microcontroller.pinとboardのマッピング状況を表示
参照: https://gist.github.com/anecdata/1c345cb2d137776d76b97a5d5678dc97
```py

import microcontroller
import board

for pin in dir(microcontroller.pin):
    if isinstance(getattr(microcontroller.pin, pin), microcontroller.Pin):
        print("".join(("microcontroller.pin.", pin, "\t")), end=" ")
        for alias in dir(board):
            if getattr(board, alias) is getattr(microcontroller.pin, pin):
                print("".join(("", "board.", alias)), end=" ")
    print()
```
### ボード名の表示
  ```py
  import os
  print(os.uname().machine)
  'Adafruit ItsyBitsy M4 Express with samd51g19'
  ```

### ひとつの`code.py`で複数のボードに対応する
  ```py
  import os
  board_type = os.uname().machine
  if 'QT Py M0' in board_type:
    tft_clk  = board.SCK
    tft_mosi = board.MOSI
    spi = busio.SPI(clock=tft_clk, MOSI=tft_mosi)
  elif 'ItsyBitsy M4' in board_type:
    tft_clk  = board.SCK
    tft_mosi = board.MOSI
    spi = busio.SPI(clock=tft_clk, MOSI=tft_mosi)
  elif 'Pico' in board_type:
    tft_clk = board.GP10 # must be a SPI CLK
    tft_mosi= board.GP11 # must be a SPI TX
    spi = busio.SPI(clock=tft_clk, MOSI=tft_mosi)
  else:
    print("supported board", board_type)
  ```


## Hacks

### REPLを使う

#### コアモジュールとライブラリの一覧を表示
  ```
  Adafruit CircuitPython 6.2.0-beta.2 on 2021-02-11; Adafruit Trinket M0 with samd21e18
  >>> help("modules")
  __main__          digitalio         pulseio           supervisor
  analogio          gc                pwmio             sys
  array             math              random            time
  board             microcontroller   rotaryio          touchio
  builtins          micropython       rtc               usb_hid
  busio             neopixel_write    storage           usb_midi
  collections       os                struct
  Plus any modules on the filesystem
  ```

#### ワンライナー

(Pythonではセミコロンで命令をつなぐことができる)

```py
# よく使われるライブラリをまとめてimportする
import time; import board; from digitalio import DigitalInOut,Pull; import analogio; import touchio

# ボードのピンとオブジェクトを表示 (例: 'I2C'、'display'など)
import board; dir(board)

# チップのピンを表示　(上記のimport boardで表示したピンとは違う)
import microcontroller; dir(microcontroller.pin)

# 内蔵ディスプレイのリリース
import displayio; displayio.release_displays()

# ボードに搭載されたNeoPixelを紫色にする
import board; import neopixel; leds = neopixel.NeoPixel(board.D3, 8, brightness=0.2); leds.fill(0xff00ff)

```

## Python情報

### コアモジュール以外のどんなライブラリがimportされているかを表示 
```py
import sys
print(sys.modules.keys())
# 'dict_keys([])'
import board
import neopixel
import adafruit_dotstar
print(sys.modules.keys())
prints "dict_keys(['neopixel', 'adafruit_dotstar'])"
```

### グローバル変数の一覧を表示
```py
a = 123
b = 'hello there'
my_globals = sorted(dir)
print(my_globals)
# prints "['__name__', 'a', 'b']"
if 'a' in my_globals:
  print("you have a variable named 'a'!")
if 'c' in my_globals:
  print("you have a variable named 'c'!")
```


## ホスト側のタスク

### CircuitPythonのライブラリをインストールする

以下はMacOS / Linuxの例です。  Windowsでも類似のコマンドを使用します。

#### `circup`でライブラリをインストールする

`circup` を使うと簡単にライブラリのインストールとアップデートができます

```sh
$ pip3 install --user circup
$ circup install adafruit_midi
$ circup update   # updates all modules
```

すべてのモジュールを最新版に更新 (例えばCircuitPython 6から7にアップデートする場合など)
(これが必要なのは、`circup update` が実際には確実に動作していないようだからです)

```sh
circup freeze > mymodules.txt
rm -rf /Volumes/CIRCUITPY/lib/*
circup install -r mymodules.txt
```


また、CircuitPythonの新しいバージョンが出たらcircupを更新します。
```sh
$ pip3 install --upgrade circup
```


#### `cp`コマンドでライブラリをコピーする

ライブラリを手動でインストールするには
[CircuitPython Library Bundle](https://circuitpython.org/libraries)
または [CircuitPython Community Bundle](https://github.com/adafruit/CircuitPython_Community_Bundle/releases) (which circup doesn't support), からバンドルをダウンロードして解凍したあと`cp -rX`を実行してください。

```sh
cp -rX bundle_folder/lib/adafruit_midi /Volumes/CIRCUITPY/lib
```

**注意：** Trinkey, Trinket, QTPyのような限られたメモリのボードでは、MacOSの`-X`オプションを使用してスペースを節約する必要があります。
また、一部のライブラリの未使用部分を削除する必要があるかもしれません（例：MIDIノートを送信するだけなら、`adafruit_midi/system_exclusive`は必要ありません）。


