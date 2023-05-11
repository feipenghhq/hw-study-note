# UPF Power Aware Verification

[TOC]

## Most Popular Power Saving Techniques

- Clock gating
- Power gating
- Multi-voltage (MV) design
- Multi-voltage + power gating
- Dynamic voltage and frequency scaling (DVFS)
- Clock teething

Power consumption:

- P_dynamic = alpha*CV<sup>2</sup>f

- P_static = V * I

- P = P_dynamic + P_static

### Clock Gating

Clock gating can reduce the **switching activity** in dynamic power. 

A lot of power are spent on clock buffer and clock has the highest switching activities.

### Power Gating

Even if clock is gated, there are still leaking current in the design which will lead to static power consumption.

Switch off the supply voltage to reduce the leaking current.

### Multi-voltage design

In multi-voltage design, depending on the SoC, different blocks may operating on different voltage. Some blocks may require higher voltage but some blocks might use lower voltage. By using different voltage, we can save power on blocks that only need lower voltage.

### Multi-voltage + power gating

Combination of power gating and multi-voltage design.

### Dynamic voltage and frequency scaling

Assume at t = t1, clock is running at f = 500MHz. If there is a heavy work load, we want to increase the clock frequency so at t = t2, f = 800MHz.

Traditionally, we will need to shut off the clock, and turned off the power. Then power will be turned on and PLL will relock to the new frequency (800MHz). There is a lost of performance during the clock frequency switching, and we also need to change frequency.

DVFS means clock will change to the new frequency without shutting down the system.

### Clocking teething

Depending on the requirement, we can skip a clock edge.



## Power Aware Verification

- Static/formal verification
- Dynamic verification
  - simulation
  - emulation
  - GLS

## Static Verification

Normal RTL simulation steps:

Linking/loading -> compilation -> elaboration -> simulation

Power aware (PA) simulation:

UPF compiled -> UPF elaboration

Then UPF and RTL are merged. UPF component such as isolation cells will be added into the design.

### Static Check Tool

- syntax and semantics
- structural
- corruption
- consistency

## Dynamic Verification

- Understand top level UPF design
  - power/voltage rail
- Control/drive voltage rails

### Controlling Power Supplies

Assume our design has 3 power rails: Vdd_h (1.2V), Vdd_l (1V), and Vss

Assume that we have a `dut.upf` for the top level design.

Assume that our test bench has the following structure.

```txt
tb
  - dut
  - tb_collaterls:
  	- bfm code
  	- top.sv
```

#### Approach #1 - Add power supply at top.sv

```verilog
top.sv:

import UPF::*;
module tb();
    reg Vdd_h, Vdd_l, Vss;
    initial begin
        #0;
        supply_on(Vss, 0.0V);
        #100;
        spply_on(Vdd_l, 1.0V);
        #100;
        supply_on(Vdd_h, 1.2V);
        #1000;
        supply_off(Vdd_h);
    end
endmodule
    
```

#### Approach #2 - Using UPF

Create a tb UPF

```txt
tb.upf:
# -> create power domain
# -> create supply port and supply net
# -> Define supply value
# -> Create Always ON Rail (Primary power of the design)
# -> create power switch
load_upf dut.upf -scope dut
```

### Sim state Modeling

### Low Power Coverage

Major power components in UPF.

- Power Domains
  - Power Supply Networks
    - Ports/Nets
    - Supply Sets
  - Power Switches
  - PST
    - add_port_state
    - create_pst
    - add_pst_state
    - add_power_state (UPF2.0+)
  - Level Shifters
  - Isolation cells
  - Retention cells

We can check coverage for the above components except for level shifter. 

Simulation can't check if level shifter is presented or not.

Isolation/Retention cells can be covered. 

### Low Power Assertions

2 categories:

- Tool/Simulator built in assertions

- User defined assertion. Such as power on sequence, power gate sequence
