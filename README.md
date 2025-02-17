# EUCLID PROBE
![IsoTN](https://github.com/user-attachments/assets/b366cb9d-9635-4fc6-a7a2-7a94b25dc28a)

The highly accurate, magneticaly coupled Z-Probe that is not affected by bed temp, bed material, magnetism or surface treatment.   

The probe can be configured to be used as Z-endstop, be manually or automatically deployed via gcode macros, and can take advantage of the firmware's probe pickup detection scheme to ensure pickup/release. It uses screw attached magnets for both mechanical coupling and for electrical contact. The Z-Probe circuit is completed when the probe is attached. 

The unique dock design securely captures the probe, providing secure and reliable docking and undocking. 

# We have moved all documentation to https://www.euclidprobe.com  
# All data and source files will remain here and be linked from here 

YouTube Channel - https://www.youtube.com/channel/UCIUXRiUfHCOrqRxitcH9O6g

Discussion and support is available as a subgroup to the CroXY Discord- https://discord.gg/jfnVrUx2uK ![croxy](images/CroXYDiscord.png)  

Parts kits and fully assembled probes are available for purchase at www.euclidprobe.com. 

The most current info, including printable dock and mount files are also hosted at www.euclidprobe.com.

**<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License</a>.**


klipper 宏示例
由于 klipper 的有机性和惯用性，没有部署和缩回探针的标准方法。因此，我们提供了一组基本的宏，作为用户自定义的框架。以下宏是基本宏，希望在调试时能打印一次。

 提示：在以下示例中，我们保持使用 M401 和 M402 进行探针部署和缩回以实现 gcode 一致性。鼓励用户修改代码以满足他们的口味和需求。
用户为各种打印机和配置提供的宏也在 klipper 文件夹中共享。这些只是为了方便起见而提供的，因此不受支持。我们尝试对它们进行命名，以便打印机位于文件名中。

给出的主要示例是固定停靠（X、Y、Z 常数），带有 Z 限位开关，以及一些示例宏。

如果需要一系列探测命令，则建议将 M401 和 M402 对括起来 gcodes 序列。请参阅 宏 示例 BED_MESH_CALIBRATE_ORIGINAL 和 HOME_LVL_MESH 宏以获取建议。

 警告：在实施第三方宏时，请特别注意在探测或探测拾取后引用 Z 行程高度或 Z 移动等内容的任何变量。
我们建议 Z 轴移动或行程高度为 15 mm，以确保探头和床之间有足够的间隙。
 提示：最新的宏在 github 存储库中作为捆绑的 .zip 文件供下载 - Euclid github 存储库
 
```
#  __________________________________________________________________________
#  |                                                                        |
#  |                                                                        |
#  |                                                                        |
#  |                                                                        |
#  |                                                                        |
#  |                                                                        |
#  |                                                                        |
#  |                                                                        |
#  |                                                                        |
#  |                                                                        |
#  |                                * Probe Ready Position                  |
#  |                                  X150 Y150                             |
#  |                                                                        |
#  |                                                                        |
#  | * Dock Re-entry staging  position                                      |
#  |   X0 Y70                                                               |
#  |                                                                        |
#  |                                                                        |
#  | * Dock Exit Position                                                   | 
#  |   X0 Y40                                                               |
#  |                                                                        |
#  |                                                                        |
#  |                                                                        |
#  |                                                                        |
#  |                                                                        |
#  |   X0 Y0    X30 Y0       X100 Y0                                        |
#  | * Dock   * Dock Side  * Dock Preflight                                 |
#  |________________________________________________________________________| 
#
# Above is example 300x300 bed to coorelate with macros and movements below.
# This example is for a fixed dock, fixed gantry/carraige and moving bed motion system. 
# RailCore, Ender5, V-Core3, etc...
# For moving gantry sytem like Voron 2.4, there are some subltle things to change.
# We have attempetd indicate those in the comments throughout the example code.   
# Z elevation is shown in movements to ensure adequate Z elevation to avoid crashes. 
# With the coupling magnets +/- 2mm of the nozzle the probe trigger height is on the
# ordeer of 12mm, so 15mm is used as a safe height.
#
# the printer.cfg-snip.txt has the following config settings 
# # 
# #...
# # ad this include statement at the head of the config file
# [include euclid.cfg]
# #
#
# #
# # enable enable_force_move to enable FORCE_MOVE and SET_KINEMATIC_POSIITION
# enable_force_move:true
# # ...
#
# It is assumed that there is a seperate Z-endstop that is used to home Z
# IF YOU ARE USING PROBE AS ENDSTOP AND PROBE homing_overide must be altered
#
# some configurtions may need FORCE_MOVE enabled for kinematic position functionS
# https://www.klipper3d.org/Config_Reference.html?h=force_move#force_move
#
# Movement Locations:
#    Users need to identify these locations and customize for their deployment: 
#       Pre-flight position X100 Y20 located to ensure clear travel path to dock
#       Dock Adjacent position X30 Y0 to provide short lateral travel for pickup and swipe off
#       Probe pickup over dock X0 Y0
#       Dock exit Position X0 Y40
#       Probe Ready Position X150 Y0 center of bed
#
# the above list of coordiantes get used in the movement macros below
#


[probe]
##    Euclid Probe
pin: ^PA0                    ; use pin PA0 and enable internal pullup resistor as this is an NC switch  use ! to invert if needed
x_offset: 2.0                ; probe is offset 2.0mm from nozzle
y_offset: 25.0               ; probe is +25mm from nozzle in Y direction
z_offset: 11.35              ; trigger point is 9.5mm below nozzle. larger numbers move effective Z0 CLOSER to the nozzle
speed: 5                     ; probing speed of 5mm/second ideal is <10mm/sec  
samples: 2                   ; number of probes to perform per sample
samples_result: average      ; normalization method: see config reference
sample_retract_dist: 3.0
samples_tolerance: 0.0075
samples_tolerance_retries: 3

#
# example homing overide to use Euclid as an endstop and Z-probe
# example assumes that the bed is 300x300
# assumes homing Z at center of bed
# assumes that macro for probe deploy and retract below are called M401 and M402
#
#
[homing_override]
gcode: SET_KINEMATIC_POSITION Z=0
 G0 Z15 F500           ; raise bed to 15
 G28 X Y               ; home Y & Y
 M401                  ; deploy Euclid Probe
 G0 X150 Y150 F6000    ; move to X150 Y150
 G28 Z                 ; home Z
 G0 Z15 F500           ; raise bed to 15
 M402                  ; retract Euclid Probe
axes: z
set_position_z: -5

# Macro to Deploy Bed Probe
[gcode_macro M401]
gcode:
    G90
    {action_respond_info("Entering M401")}
    error_if_probe_deployed    ; check to make sure that the probe is not already attached
    _M401

[gcode_macro error_if_probe_deployed]
gcode:
    QUERY_PROBE                 ; check probe status
    do_error_if_probe_deployed  ; logic check to verify probe is not already deployed

[gcode_macro do_error_if_probe_deployed]
gcode:
    {% if not printer.probe.last_query %}
      {action_raise_error("Euclid Probe is already deployed - Remove and Return it to the dock")}
    {% endif %}

# Macro to Deploy Bed Probe
[gcode_macro _M401]
gcode:
    G90
    {% if printer.probe.last_query %} 
      G0 Z15 F3000         ;  set approach elevation of Z15 to clear probe over bed on fixed gantry machine
      #                       for moving gantry machine this may need to be adjusted
      G0 X100 Y0          ;  move the carraige to safe position to move from
      G0 X30 Y0            ;  move to the side of the dock
      G4 P250              ;  wait 1/4 second 
      G0 X0 Y0             ;  move sideways over the dock to pick up probe
      M400                 ;  wait for moves to finish
      G4 P250              ;  pause 1/4 sec for detection
      G0 X0 Y40            ;  move out of the dock in a straight line 
      G0 Z15               ;  move up to clear the probe over the bed of moving gantry or provide clearance for fixed gantry
      G0 X150 Y150         ;  move probe to center of bed in ready position 
    {% endif %}
    error_if_probe_not_deployed
    {action_respond_info("Exiting M401")}

[gcode_macro error_if_probe_not_deployed]
gcode:
    QUERY_PROBE
    do_error_if_probe_not_deployed

[gcode_macro do_error_if_probe_not_deployed]
gcode:
    {% if printer.probe.last_query %}
      {action_raise_error("Euclid Probe failed to deploy!")}
    {% endif %}

# Macro to retract Bed Probe
[gcode_macro M402]
gcode:
    G90
    {action_respond_info("Entering M402")}
    error_if_probe_not_deployed
    _M402

# Macro to Stow Bed Leveling Probe
[gcode_macro _M402]
gcode:
    G90
    {% if not printer.probe.last_query %} ; the logic on this needs function check
      G0 Z15 F2400                  ;  set approach elevation of Z15 for fixed gantry system to clear probe over bed
      #                             ;  for moving gantry system this may have to be altered to match your dock elevation
      G0 X150 Y150 F3000            ;  start movements at center of the bed 
      G0 X0 Y70 F3000               ;  move to the re-entry staging position
      G0 X0 Y40 F3000               ;  move to a position in front of the dock so simple linear movement into dock 
      G0 X0 Y0 F240                 ;  slowly move into dock 
      M400                          ;  wait for moves to finish
      G4 P250                       ;  forced pause here so motion is definite 90 tavel to swipe
      G0 X30 Y0 F6000               ;  quick swipe off 
      G0 X150 Y0                    ;  move to front center of bed                   
      G0 Z20 F500                   ;  move up to elevation of Z20
    {% endif %}                     ;  exit the if-then loop. was missing in previous versions
    error_if_probe_deployed         ;  verify that the probe is detached. is corrected error  
    {action_respond_info("Exiting M402")}


# Macro to perform a bed mesh calibration by wrapping it between M401/M402 macros
[gcode_macro BED_MESH_CALIBRATE]
rename_existing:    BED_MESH_CALIBRATE_ORIGINAL
gcode:
  M401                           ; deploy Euclid Probe if needed
  BED_MESH_CALIBRATE_ORIGINAL    ; check bed level
  M402                           ; dock Euclid Probe

# Macro to perform a modified z_tilt  by wrapping it between M401/M402 macros
[gcode_macro QUAD_GANTRY_LEVEL]
rename_existing:    QUAD_GANTRY_LEVEL_ORIGINAL
gcode:
  M401                           ; deploy Euclid Probe if needed
  _QUAD_GANTRY_LEVEL_ORIGINAL         ; check bed level
  M402                           ; dock Euclid Probe

[gcode_macro HOME_LVL_MESH]
gcode: gcode: SET_KINEMATIC_POSITION Z=0
  G0 Z15 F500           ; raise bed to 15
  G28 X Y               ; home Y & Y
  M401                  ; deploy Euclid Probe
  G0 X150 Y150 F6000    ; move to center of be @ X150 Y150
  G28 Z                 ; home Z
  QUAD_GANTRY_LEVEL_ORIGINAL
  G28 Z                 ; home Z
  BED_MESH_CALIBRATE_ORIGINAL
  G0 Z15 F500           ; raise bed to 15
  M402                  ; retract Euclid Probe
```
