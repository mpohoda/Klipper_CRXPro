# Klipper Install and Config for Creality CR-X Pro

## Install
Download latest RPi imager:

[for Windows](https://downloads.raspberrypi.org/imager/imager_latest.exe)

[for MacOS](https://downloads.raspberrypi.org/imager/imager_latest.dmg)

Choose OS > Other specific-purpose OS > 3D printing > Mainsail OS

Choose Storage (SD Card)

Setup username, password, wifi, keyboard layout, etc.

**Write** to card

## Make & Flash
Go to RPi command line via Putty:

[latest](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

```
cd ~/klipper
make menuconfig
```

Setup as in picture, then Q(uit) and Y(es) for save:

![image](https://user-images.githubusercontent.com/33594918/225866672-06355f93-8259-43a6-a8e6-abf712fa1a48.png)
```
ls /dev/serial/by-id/*
```
as output you get someting like this: `/dev/serial/by-id/usb-FTDI_FT232R_USB_UART_A10K2GAF-if00-port0`

and put it after `make flash FLASH_DEVICE=`

```
sudo service klipper stop
make flash FLASH_DEVICE=/dev/serial/by-id/usb-FTDI_FT232R_USB_UART_A10K2GAF-if00-port0
sudo service klipper start
```

# Configure
printer.cfg for Creality CR-X Pro (v.1 - some unsupported values)
```
[mcu]
serial: /dev/serial/by-id/usb-FTDI_FT232R_USB_UART_A10K2GAF-if00-port0
 
##---- this config, the firmware should be compiled for the AVR atmega2560.
 
[stepper_x]
step_pin: PF0
dir_pin: PF1
enable_pin: !PD7
microsteps: 16
rotation_distance: 40
endstop_pin: ^PE5
position_endstop: 0
position_max: 300
homing_speed: 50
 
[stepper_y]
step_pin: PF6
dir_pin: !PF7
enable_pin: !PF2
microsteps: 16
rotation_distance: 40
endstop_pin: ^PJ1
position_endstop: 0
position_max: 310
homing_speed: 50
 
[stepper_z]
step_pin: PL3
dir_pin: !PL1
enable_pin: !PK0
microsteps: 16
rotation_distance: 8
 
##-----comment out next line if you do not have a BLTouch
endstop_pin: probe:z_virtual_endstop
 
##-----uncomment next two lines if you do not have a BLTouch
#endstop_pin: ^PD3
#position_endstop: 10
 
position_max: 400
position_min: -6.0
 
[extruder]
step_pin: PA4
dir_pin: PA6
enable_pin: !PA2
microsteps: 16
rotation_distance: 23.072855
nozzle_diameter: 0.400
filament_diameter: 1.750
heater_pin: PB4
sensor_type: EPCOS 100K B57560G104F
sensor_pin: PK5
control: pid
pid_Kp: 22.2
pid_Ki: 1.08
pid_Kd: 114
min_temp: 0
max_temp: 265
pressure_advance: 0.155
 
 
[extruder1]
step_pin: PC1
dir_pin:  PC3
enable_pin: !PC7
microsteps: 16
rotation_distance: 23.072855
nozzle_diameter: 0.400
filament_diameter: 1.750 
shared_heater: extruder
pressure_advance: 0.155
 
 
[heater_bed]
heater_pin: PH5
sensor_type: ATC Semitec 104GT-2
sensor_pin: PK6
control: pid
pid_Kp: 690.34
pid_Ki: 111.47
pid_Kd: 1068.83
min_temp: 0
max_temp: 130
 
[fan]
pin: PH6
 
[safe_z_home]
home_xy_position: 150, 150
speed: 100
z_hop: 10
z_hop_speed: 5
 
##-------comment out entire [bltouch] section if you do not have a BLTouch
[bltouch]
sensor_pin: ^PD3
control_pin: PB5
x_offset: 55
y_offset: -6
z_offset: 7
samples: 2
speed: 6.0
pin_up_touch_mode_reports_triggered: False
 
## -----These settings need to be updated based on your BLTouch mounting position
[bed_mesh]
speed: 180
horizontal_move_z: 5
mesh_min: 60,6
mesh_max: 290,304
probe_count: 5,5
fade_start: 1.0
fade_end: 10.0
#mesh_pps: 2, 2
algorithm: bicubic
relative_reference_index: 24
 
[bed_screws]
screw1: 25, 50
screw2: 265, 50
screw3: 265, 270
screw4: 25, 270
 
[printer]
kinematics: cartesian
max_velocity: 300
max_accel: 3000
max_z_velocity: 5
max_z_accel: 100
```


Cura Start G-Code
```
SET_GCODE_VARIABLE MACRO=START_PRINT VARIABLE=bed_temp VALUE={material_bed_temperature_layer_0}
SET_GCODE_VARIABLE MACRO=START_PRINT VARIABLE=extruder_temp VALUE={material_print_temperature_layer_0}
START_PRINT
```

Cura End G-Code
```
SET_GCODE_VARIABLE MACRO=END_PRINT VARIABLE=machine_depth VALUE={machine_depth}
END_PRINT
```

PrusaSlicer Start G-Code
```
SET_GCODE_VARIABLE MACRO=START_PRINT VARIABLE=bed_temp VALUE={first_layer_bed_temperature[0]}
SET_GCODE_VARIABLE MACRO=START_PRINT VARIABLE=extruder_temp VALUE={first_layer_temperature[0]}
START_PRINT
```

PrusaSlicer End G-Code
```
SET_GCODE_VARIABLE MACRO=END_PRINT VARIABLE=machine_depth VALUE={print_bed_max[1]}
END_PRINT
```
### ADXL345 setup
Wiring diagram

![image](https://user-images.githubusercontent.com/33594918/225709045-1f259db4-2a5b-437c-98df-115e01f85e7c.png)

Software install via Putty
```
sudo apt update
sudo apt install python3-numpy python3-matplotlib libatlas-base-dev
~/klippy-env/bin/pip install -v numpy
```

Install rc script
```
cd ~/klipper/
sudo cp ./scripts/klipper-mcu.service /etc/systemd/system/
sudo systemctl enable klipper-mcu.service
```

Building the micro-controller code
```
cd ~/klipper/
make menuconfig
```

![image](https://user-images.githubusercontent.com/33594918/225734204-0951a155-3fd2-49bb-aa9d-1a9856ce0770.png)

```
sudo service klipper stop
make flash
sudo service klipper start
```

The following command will add the "pi" user to the tty group: (change "pi" to your real user)
```
sudo usermod -a -G tty pi
```

Optional: Identify the correct gpiochip
```
sudo apt-get install gpiod
gpiodetect
gpioinfo
```

## Plugins
### Telegram-Bot
```
cd ~
git clone https://github.com/nlef/moonraker-telegram-bot.git
cd moonraker-telegram-bot
./scripts/install.sh
```

Create telegram bot

This is done by talking to [BotFather](https://telegram.me/botfather) on telegram. Name the bot, give it a username. You will get a token to control the bot.

edit telegram.conf
```
#  Please refer to the wiki(https://github.com/nlef/moonraker-telegram-bot/wiki) for detailed information on how to configure the bot

[bot]
server: localhost
bot_token: add_your_token_here
chat_id: add_your_chat-id_here

[camera]
host: http://localhost:8080/?action=stream

[progress_notification]
percent: 5
height: 5
time: 5

#[timelapse]
#cleanup: true
#height: 0.2
#time: 5
#target_fps: 30
```

add to moonraker.conf
```
[update_manager client moonraker-telegram-bot]
type: git_repo
path: ~/moonraker-telegram-bot
origin: https://github.com/nlef/moonraker-telegram-bot.git
env: ~/moonraker-telegram-bot-env/bin/python
requirements: scripts/requirements.txt
install_script: scripts/install.sh
```
### Shell Commands
```
cd ~
git clone https://github.com/th33xitus/kiauh.git
./kiauh/kiauh.sh
```

![image](https://user-images.githubusercontent.com/33594918/225716494-316f0b8e-5122-4d17-9275-6058a1ae5968.png)

[4] Advanced > Extensions > [8 or 9] Shell Command

edit shell_command.cfg
```
# example
[gcode_shell_command print_to_file]
command: sh /home/pi/klipper_config/script.sh
timeout: 30.
verbose: True

[gcode_macro GET_TEMP]
gcode:
    {% set temp = printer.extruder.temperature %}
    { action_respond_info("%s" % (temp)) }
    RUN_SHELL_COMMAND CMD=print_to_file PARAMS={temp}
```

### Mobileraker
```
cd ~/
git clone https://github.com/Clon1998/mobileraker_companion.git
cd mobileraker_companion
./scripts/install-mobileraker-companion.sh
```

edit mobileraker.conf
```
[general]
language: en 
# one of the supported languages defined in i18n.py#languages (de,en,...)
# Default: en
timezone: Europe/Berlin 
# correct timezone e.g. Europe/Berlin for Berlin time or US/Central. 
# For more values see https://gist.github.com/heyalexej/8bf688fd67d7199be4a1682b3eec7568
# Default: Tries to use system timezone
# Optional
eta_format: %%d.%%m.%%Y, %%H:%%M:%%S
# Format used for eta and adaptive_eta placeholder variables
# For available options see https://strftime.org/
# Note that you will have to escape the % char by using a 2nd one e.g.: %d/%m/%Y -> %%d/%%m/%%Y
# Default: %%d.%%m.%%Y, %%H:%%M:%%S
# Optional

# Add a [printer ...] section for every printer you want to add
[printer <NAME OF YOUR PRINTER: optional>]
moonraker_uri: ws://127.0.0.1:7125/websocket
# Define the uri to the moonraker instance.
# Default value: ws://127.0.0.1:7125/websocket
# Optional
moonraker_api_key: False
# Moonraker API key if force_logins or trusted clients is active!
# Default value: False
# Optional
```

add to moonraker.conf
```
[update_manager mobileraker]
type: git_repo
path: ~/mobileraker_companion
origin: https://github.com/Clon1998/mobileraker_companion.git
primary_branch:main
managed_services: mobileraker
env: ~/mobileraker-env/bin/python
requirements: scripts/mobileraker-requirements.txt
install_script: scripts/install-mobileraker-companion.sh
```


# References
Creality Sonic Pad - Printer Config for Ender 3 S1 Pro [STM32F401]

https://github.com/CrealityOfficial/Creality_Sonic_Pad/blob/main/printer_configrations/


How to Install MainsailOS on Raspberry Pi

https://3dprintbeginner.com/how-to-install-mainsailos-on-raspberry-pi/


Measuring Resonances via ADXL345

https://www.klipper3d.org/Measuring_Resonances.html


RPi microcontroller for linux (ADXL345)

https://www.klipper3d.org/RPi_microcontroller.html
