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
