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
or
```
sudo service klipper stop
sudo avrdude -v -p atmega2560 -c wiring -P /dev/serial/by-id/usb-FTDI_FT232R_USB_UART_A10K2GAF-if00-port0 -b 115200 -D -U flash:w:out/klipper.elf.hex:i
sudo service klipper start
```
you can try also different baud rate e.g. `-b 57600` for some boards.

# Configure
printer.cfg (for Creality CR-X Pro - v.1 some unsupported values)
```
################## Creality CR-X Pro Klipper Config ###############
# !Creality CR-X Pro
# printer_size: 300x300x400
# To use this config, during "make menuconfig" select the AVR atmega2560

# Flash this firmware by "make flash FLASH_DEVICE=/dev/serial/by-id/usb-FTDI_FT232R_USB_UART_A10K2GAF-if00-port0"

# See docs/Config_Reference.md for a description of parameters.

[include mainsail.cfg]
[include shell_command.cfg]

[mcu]
serial: /dev/serial/by-id/usb-FTDI_FT232R_USB_UART_A10K2GAF-if00-port0
 
[mcu rpi]
serial: /tmp/klipper_host_mcu

[adxl345]
cs_pin: rpi:None

[resonance_tester]
accel_chip: adxl345
probe_points:
    100, 100, 20  # an example
 
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
z_offset: 0
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

[temperature_sensor Raspberry_Pi]
sensor_type: temperature_host
min_temp: 0
max_temp: 100

[virtual_sdcard]
path: ~/printer_data/gcodes

[display_status]

[pause_resume]
recover_velocity: 25

[gcode_macro PAUSE]
description: Pause the actual running print
rename_existing: PAUSE_BASE
### change this if you need more or less extrusion ###
variable_extrude: 1.0
gcode:
  ##### read E from pause macro #####
  {% set E = printer["gcode_macro PAUSE"].extrude|float %}
  ##### set park positon for x and y #####
  # default is your max posion from your printer.cfg
  {% set x_park = printer.toolhead.axis_maximum.x|float - 5.0 %}
  {% set y_park = printer.toolhead.axis_maximum.y|float - 5.0 %}
  ##### calculate save lift position #####
  {% set max_z = printer.toolhead.axis_maximum.z|float %}
  {% set act_z = printer.toolhead.position.z|float %}
  {% if act_z < (max_z - 2.0) %}
      {% set z_safe = 2.0 %}
  {% else %}
      {% set z_safe = max_z - act_z %}
  {% endif %}
  ##### end of definitions #####
  PAUSE_BASE
  G91
  {% if printer.extruder.can_extrude|lower == 'true' %}
    G1 E-{E} F2100
  {% else %}
    {action_respond_info("Extruder not hot enough")}
  {% endif %}
  {% if "xyz" in printer.toolhead.homed_axes %}
    G1 Z{z_safe} F900
    G90
    G1 X{x_park} Y{y_park} F6000
  {% else %}
    {action_respond_info("Printer not homed")}
  {% endif %} 

[gcode_macro RESUME]
description: Resume the actual running print
rename_existing: RESUME_BASE
gcode:
  ##### read E from pause macro #####
  {% set E = printer["gcode_macro PAUSE"].extrude|float %}
  #### get VELOCITY parameter if specified ####
  {% if 'VELOCITY' in params|upper %}
    {% set get_params = ('VELOCITY=' + params.VELOCITY)  %}
  {%else %}
    {% set get_params = "" %}
  {% endif %}
  ##### end of definitions #####
  {% if printer.extruder.can_extrude|lower == 'true' %}
    G91
    G1 E{E} F2100
  {% else %}
    {action_respond_info("Extruder not hot enough")}
  {% endif %}  
  RESUME_BASE {get_params}

[gcode_macro CANCEL_PRINT]
description: Cancel the actual running print
rename_existing: CANCEL_PRINT_BASE
gcode:
  TURN_OFF_HEATERS
  {% if "xyz" in printer.toolhead.homed_axes %}
    G91
    G1 Z4.5 F300
    G90
  {% else %}
    {action_respond_info("Printer not homed")}
  {% endif %}
    G28 X Y
  {% set y_park = printer.toolhead.axis_maximum.y|float - 5.0 %}
    G1 Y{y_park} F2000
    M84
  CANCEL_PRINT_BASE

[gcode_macro M75]
gcode:

[gcode_macro M77]
gcode:

[gcode_macro G29]
gcode:
  G28
  bed_mesh_calibrate
  G1 X0 Y0 Z10 F4200
  # save_config

[gcode_macro START_PRINT]
variable_bed_temp: 60
variable_extruder_temp: 185
gcode:
    # Turn ON Power for DRYER
    POWER_ON_DRYER
    # Start bed heating
    M140 S{bed_temp}
    # Use absolute coordinates
    G90
    # Reset the G-Code Z offset (adjust Z offset if needed)
    ###SET_GCODE_OFFSET Z=0
    # Home the printer
    G28
    # Load bed level
    M420
    # Move the nozzle near the bed
    G1 Z10 F3000
    # Move the nozzle very close to the bed
    ###G1 Z0.15 F300
    # Wait for bed to reach temperature
    M190 S{bed_temp}
    # Set and wait for nozzle to reach temperature
    M109 S{extruder_temp}
    # Reset Extruder
    G92 E0
    # Move Z Axis up
    G1 Z2.0 F3000
    # Move to start position
    G1 X2.1 Y20 Z0.28 F5000.0
    # Draw the first line
    G1 X2.1 Y200.0 Z0.28 F1500.0 E15
    # Move to side a little
    G1 X2.4 Y200.0 Z0.28 F5000.0
    # Draw the second line
    G1 X2.4 Y20 Z0.28 F1500.0 E30
    # Reset Extruder
    G92 E0
    # Move Z Axis up
    G1 Z2.0 F3000
    # Print message on LCD
    M117 Printing...

[gcode_macro END_PRINT]
variable_machine_depth: 235
gcode:
    # Turn off bed, extruder, and fan
    M140 S0
    M104 S0
    M106 S0
    # Relative positionning
    G91
    # Retract a bit
    G1 E-2 F2700
    # Retract and raise Z
    G1 Z0.2 E-2 F2400
    # Wipe out
    G1 X5 Y5 F3000
    # Raise Z more
    G1 Z10
    # Absolute positionning
    G90
    # Present print
    G1 X0 Y{machine_depth}
    # Disable steppers
    M84 X Y E ;Disable all steppers but Z
    POWER_OFF_DRYER ;Turn OFF Power for DRYER
    UPDATE_DELAYED_GCODE ID=POWER_OFF_CHECK DURATION=30
    M117 Finished! Cooling down under 50°C before POWER OFF...

[gcode_macro M420]
gcode:
  BED_MESH_PROFILE LOAD=default
  
[delayed_gcode bed_mesh_init]
initial_duration: .01
gcode:
  BED_MESH_PROFILE LOAD=default

[gcode_macro AUTO_CALIBRATE_Y]
gcode:
  G28
  SHAPER_CALIBRATE AXIS=Y
  SAVE_CONFIG

[gcode_macro AUTO_CALIBRATE_X]
gcode:
  G28
  SHAPER_CALIBRATE AXIS=X
  SAVE_CONFIG

[delayed_gcode POWER_OFF_CHECK]
gcode:
  {% if printer.idle_timeout.state == "Idle" or printer.idle_timeout.state == "Ready" %}
    {% if printer.extruder.temperature < 60.0 and printer.heater_bed.temperature < 50.0 %}
        {% if printer.extruder.target == 0.0 and printer.heater_bed.target == 0.0 %}
            UPDATE_DELAYED_GCODE ID=POWER_OFF_CHECK DURATION=0
            POWER_OFF ;Turn OFF Power for printer
        {% else %}
            UPDATE_DELAYED_GCODE ID=POWER_OFF_CHECK DURATION=2
        {% endif %}
    {% else %}
        {% if printer.idle_timeout.state == "Printing" %}
            UPDATE_DELAYED_GCODE ID=POWER_OFF_CHECK DURATION=0
        {% else %}
            {% if printer.extruder.target == 0.0 and printer.heater_bed.target == 0.0 %}
                UPDATE_DELAYED_GCODE ID=POWER_OFF_CHECK DURATION=2
            {% else %}
                UPDATE_DELAYED_GCODE ID=POWER_OFF_CHECK DURATION=0
            {% endif %}
        {% endif %}
    {% endif %}
  {% endif %}

```

printer.cfg (for Creality CR-X Pro - v.2) this config was created during several prints and also after calibrations with sensors and test prints 

```
# !Creality CR-X Pro
# printer_size: 300x300x400
# To use this config, during "make menuconfig" select the AVR atmega2560

# Flash this firmware by "make flash FLASH_DEVICE=/dev/serial/by-id/usb-FTDI_FT232R_USB_UART_A10K2GAF-if00-port0"

# See docs/Config_Reference.md for a description of parameters.

[include mainsail.cfg]
[include shell_command.cfg]

[mcu]
serial: /dev/serial/by-id/usb-FTDI_FT232R_USB_UART_A10K2GAF-if00-port0

[mcu rpi]
serial: /tmp/klipper_host_mcu

[adxl345]
cs_pin: rpi:None

[resonance_tester]
accel_chip: adxl345
probe_points:
    100, 100, 20  # an example
 
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

#input shaper in printer config after measurment with sensor these values are some defaults for CR family printers working also for our CR-X Pro
#CR10S
#[input_shaper]
#shaper_freq_x: 91.3
#shaper_type_x: ei
#shaper_freq_y: 42.6
#shaper_type_y: ei

#CR10
#shaper_freq_x: 104.0
#shaper_type_x: mzv
#shaper_freq_y: 28.6
#shaper_type_y: mzv

#CR10V3
#shaper_freq_x: 39.0
#shaper_type_x: 2hump_ei
#shaper_freq_y: 39.0
#shaper_type_y: 2hump_ei

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
pressure_advance: 0.388
max_extrude_only_distance: 1000
 
[extruder_stepper extruder1]
extruder: extruder
step_pin: PC1
dir_pin:  PC3
enable_pin: !PC7
microsteps: 16
rotation_distance: 23.072855

[verify_heater extruder]  
heating_gain: 2 
check_gain_time:35  
hysteresis: 10  
max_error: 285

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
#z_offset: 0.0
samples: 2
speed: 6.0
pin_up_touch_mode_reports_triggered: False
 
## -----These settings need to be updated based on your BLTouch mounting position
[bed_mesh]
speed: 180
horizontal_move_z: 5
mesh_min: 60,6
mesh_max: 290,304
probe_count: 10,10
fade_start: 1.0
fade_end: 10.0
mesh_pps: 2, 2
algorithm: bicubic
relative_reference_index: 24
 
[bed_screws]
screw1: 25, 50
screw2: 265, 50
screw3: 265, 270
screw4: 25, 270

[printer]
kinematics: cartesian
#max_velocity: 300
max_velocity: 500
#max_accel: 3000
max_accel: 5000
max_accel_to_decel: 3000
max_z_velocity: 5
max_z_accel: 100

#[firmware_retraction]
#retract_length: 6.5
#retract_speed: 60
#unretract_extra_length: 0
#unretract_speed: 60

[temperature_sensor Raspberry_Pi]
sensor_type: temperature_host
min_temp: 0
max_temp: 100

[virtual_sdcard]
path: ~/gcode_files

[display_status]

[pause_resume]

#telegram response setup
[respond]
#default_type: echo
#   Sets the default prefix of the "M118" and "RESPOND" output to one
#   of the following:
#       echo: "echo: " (This is the default)
#       command: "// "
#       error: "!! "
#default_prefix: echo:
#   Directly sets the default prefix. If present, this value will
#   override the "default_type"

[gcode_macro CALIBRATE_X]
gcode:
  G28
  SHAPER_CALIBRATE AXIS=X
  SAVE_CONFIG

[gcode_macro CALIBRATE_Y]
gcode:
  G28
  SHAPER_CALIBRATE AXIS=Y
  SAVE_CONFIG

[gcode_macro telegram_test]
gcode:
    RESPOND PREFIX=tgnotify MSG="test"
  
# --------------------------- Start Print ----------------------------
[gcode_macro START_PRINT]
variable_bed_temp: 60
variable_extruder_temp: 185
gcode:
    # Start bed heating
    M140 S{bed_temp}
    # Use absolute coordinates
    G90
    # Reset the G-Code Z offset (adjust Z offset if needed)
    SET_GCODE_OFFSET Z=0.0
    # Home the printer
    G28
    # Move the nozzle near the bed
    G1 Z5 F3000
    # Move the nozzle very close to the bed
    #G1 Z0.15 F300
    # Wait for bed to reach temperature
    M190 S{bed_temp}
    # Set and wait for nozzle to reach temperature
    M109 S{extruder_temp}
    # Reset Extruder
    G92 E0
    # Move Z Axis up
    G1 Z2.0 F3000
    # Move to start position
    G1 X2.1 Y20 Z0.28 F5000.0
    # Draw the first line
    G1 X2.1 Y200.0 Z0.28 F1500.0 E15
    # Move to side a little
    G1 X2.4 Y200.0 Z0.28 F5000.0
    # Draw the second line
    G1 X2.4 Y20 Z0.28 F1500.0 E30
    # Reset Extruder
    G92 E0
    # Move Z Axis up
    G1 Z2.0 F3000
    # Print message on LCD
    M117 By your command!
# --------------------------------------------------------------------

# ---------------------------- End Print -----------------------------
[gcode_macro END_PRINT]
variable_machine_depth: 235
gcode:
    # Turn off bed, extruder, and fan
    M140 S0
    M104 S0
    M106 S0
    # Relative positionning
    G91
    # Retract and raise Z
    G1 Z0.2 E-2 F2400
    # Wipe out
    G1 X5 Y5 F3000
    # Raise Z more
    G1 Z10
    #retract filament 100mm
    G92 E0
    G1 F2000 E-100  
    G92 E0
    # Absolute positionning
    G90
    # Present print
    G1 X0 Y{machine_depth}
    # Disable steppers
    M84
    # Print message on LCD
    M117 That's All Folks
# --------------------------------------------------------------------

[gcode_macro T0]
gcode:
    SET_GCODE_OFFSET X=0 Y=0
    SYNC_STEPPER_TO_EXTRUDER STEPPER=extruder1 EXTRUDER=
    SYNC_STEPPER_TO_EXTRUDER STEPPER=extruder EXTRUDER=extruder

[gcode_macro T1]
gcode:
    SET_GCODE_OFFSET X=0 Y=0
    SYNC_STEPPER_TO_EXTRUDER STEPPER=extruder EXTRUDER=
    SYNC_STEPPER_TO_EXTRUDER STEPPER=extruder1 EXTRUDER=extruder

[gcode_macro ACTIVATE_EXTRUDER]
description: Replaces built-in macro for a X-in, 1-out extruder configuration SuperSlicer fix
rename_existing: ACTIVATE_EXTRUDER_BASE
gcode:
    {% if 'EXTRUDER' in params %}
      {% set ext = params.EXTRUDER|default(EXTRUDER) %}
      {% if ext == "extruder"%}
        {action_respond_info("Switching to extruder0.")}
        T0
      {% elif ext == "extruder1" %}
        {action_respond_info("Switching to extruder1.")}
        T1
      {% else %}
        {action_respond_info("EXTRUDER value being passed.")}
        ACTIVATE_EXTRUDER_BASE EXTRUDER={ext}
      {% endif %}
    {% endif %}

[delayed_gcode activate_default_extruder]
initial_duration: 1
gcode:
    ACTIVATE_EXTRUDER EXTRUDER=extruder

[delayed_gcode bed_mesh_init]
initial_duration: .01
gcode:
  BED_MESH_PROFILE LOAD=default

[gcode_macro M104]
description: Replaces built-in gcode to not specify Tx due to single extruder
rename_existing: M104.1
gcode:
    {% set s = params.S|default(0)|float %}
    {% set t = params.T|default(0)|int %}
    {% if 'S' in params %}
      {%  if 'T' in params %}
        M104.1 S{s}
      {% else %}
        M104.1 S{s}
      {% endif %}
    {% endif %}

[gcode_macro M205]
gcode:
  {% if 'X' in params %}
    SET_VELOCITY_LIMIT SQUARE_CORNER_VELOCITY={params.X}
  {% elif 'Y' in params %}
    SET_VELOCITY_LIMIT SQUARE_CORNER_VELOCITY={params.Y}
  {% endif %}

#*# <---------------------- SAVE_CONFIG ---------------------->
#*# DO NOT EDIT THIS BLOCK OR BELOW. The contents are auto-generated.
#*#
#*# [bltouch]
#*# z_offset = 3.265
#*#
#*# [bed_mesh default]
#*# version = 1
#*# points =
#*# 	0.012500, -0.015000, -0.031250, -0.045000, 0.017500
#*# 	0.037500, -0.026250, -0.065000, -0.070000, -0.010000
#*# 	0.070000, 0.010000, -0.035000, -0.050000, 0.005000
#*# 	0.022500, -0.030000, -0.066250, -0.071250, -0.016250
#*# 	0.010000, -0.017500, -0.040000, -0.046250, 0.000000
#*# x_count = 5
#*# y_count = 5
#*# mesh_x_pps = 2
#*# mesh_y_pps = 2
#*# algo = bicubic
#*# tension = 0.2
#*# min_x = 60.0
#*# max_x = 290.0
#*# min_y = 6.0
#*# max_y = 304.0
#*#
#*# [input_shaper]
#*# shaper_type_x = mzv
#*# shaper_freq_x = 85.6
#*# shaper_type_y = mzv
#*# shaper_freq_y = 30.8
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
[printer]
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
Creality Sonic Pad - Printer Configs (e.g. CR10S)

https://github.com/CrealityOfficial/Creality_Sonic_Pad/blob/main/printer_configrations/


How to Install MainsailOS on Raspberry Pi

https://3dprintbeginner.com/how-to-install-mainsailos-on-raspberry-pi/


Measuring Resonances via ADXL345

https://www.klipper3d.org/Measuring_Resonances.html


RPi microcontroller for linux (ADXL345)

https://www.klipper3d.org/RPi_microcontroller.html
