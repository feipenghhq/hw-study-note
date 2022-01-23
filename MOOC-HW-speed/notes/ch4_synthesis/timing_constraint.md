# Timing Constraint

- [Timing Constraint](#timing-constraint)
  - [Constraint Area](#constraint-area)
  - [Constraint Timing](#constraint-timing)
  - [DRC (rule) constraint](#drc-rule-constraint)
  - [Example](#example)

## Constraint Area

Command: `set_max_area 100`

Units are those of target library, defined by the vendor. Can be the following:

- 2-input-NAND-gate
- number of transistors
- square mils

Tips to get the unit: Synthesis a 2-input NAND gate and use `report_area` to get the area. If the result is:

- 1 => defined as 2-input-NAND-gate
- 4 => defined as number of transistors
- other value => defined as square mils

## Constraint Timing

The timing path can be divided into 4 categories:

- Input to register
- Register to Register
- Register to output
- Input to output

### Defining clocks (register to register)

Value user must define: Clock source (port or pin), Clock period

Value user may define (optional): Duty cycle, Offset/Skew, Clock name

Command:

```tcl
create_clock -period 10 [get_ports Clk]
set_dont_touch_network [get_clocks Clk]
```

Set all the clock network as dont_touch, so synthesis does not optimize clock signal. Optimization will be done at CTS

### Defining input delay

Input delay is the time that a signal spend outside the design being synthesis to reach the design.

Command:

```tcl
set_input_delay -max 4 -clock Clk [get_ports A]
```

-max is used to constraint setup time. -min is used to constraint hold time.

### Defining output delay

Similar to input delay.

Command:

```tcl
set_output_delay -max 4 -clock Clk [get_ports A]
```

## DRC (rule) constraint

### set_max_transition

Used to constraint the maximum transition time for a signal in the design. The transition time depends on the load of the signal, the more the load, the more the transition time.

Can be divided to data transition time and clock transition time. Clock transition time should be smaller.

### set_max_fanout

Used to constraint the fanout of net, output, and port.

### set_max_capacitance

Defined based on the technology file

## Example

```tcl
sh date                 # show start time
remove_design -designs  # remove the original design in dc

# Set library

set search_path [list <some paths>]
set target_library {tt.db}
set link_library {* tt.db}
set symbol_library {tt.sdb}

# Read, link, check design

read_file -format verilog ~/example1.v

#analyze -format verilog ~/example1.v
#elaborate EXAMPLE1

current_design EXAMPLE1 # set EXAMPLE1 as the top level module
uniquify
check_design

# Define IO port name

set clk [get_ports clk]
set rst_n [get_ports rst_n]

set general_inputs [list a b c]
set outputs [get_ports o]

# set constraints

create_clock -n clock $clk -period 20 -waveform {0 10}
set_dont_touch_network [get_clocks clock]
set_drive 0 $clk                    # set the driver strength as infinite for clock port
set_ideal_network [get_ports clk]   # set clock ports as ideal network

set_dont_touch_network $rst_n
set_drive 0 $rst_n
set_ideal_network [get_ports rst_n]

set_input_delay -clock clock 8 $general_inputs
set_output_delay -clock clock 8 $outputs

set_max_fanout 4 $general_inputs
set_max_transition 0.5 [get_design "EXAMPLE1"]

set_max_area 0  # make the circuit as small as possible with timing constraint.

```