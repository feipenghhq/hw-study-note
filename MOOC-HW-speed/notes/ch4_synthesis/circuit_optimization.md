# Circuit Optimization

- [Circuit Optimization](#circuit-optimization)
  - [Introduction](#introduction)
  - [Create Path Group](#create-path-group)
  - [Optimizing near-critical paths](#optimizing-near-critical-paths)
  - [Performing High-Effort Compile](#performing-high-effort-compile)
  - [Performing High-Effort Compile Incremental Compile](#performing-high-effort-compile-incremental-compile)
  - [Gate-Level Optimization](#gate-level-optimization)
  - [Automatic ungrouping](#automatic-ungrouping)
  - [Adaptive retiming](#adaptive-retiming)
  - [High-level optimization and datapath optimization](#high-level-optimization-and-datapath-optimization)
  - [Verifying Functional Equivalence](#verifying-functional-equivalence)
  - [Partitioning for Synthesis](#partitioning-for-synthesis)

## Introduction

DC optimize the design to balance timing and area to satisfy the requirement for timing, power, and area.

The optimization process is based on the constraint:

- design rule constraint: transition, fanout, capacitance
- optimization constraint: delay, area

By default, DRC constraint has higher priority than optimization constraint.

## Create Path Group

By default, DC make path group based on timing. But if the timing or timing constraint is complex in the design, user can group the critical path so DC can focus on optimizing them.

User can also give priority to different group. Priority value are from 0.0 to 100.0.

Example: ```group_path -name group3 -from in3 to FF1/D -weight 2.5```

## Optimizing near-critical paths

By default, DC will only optimize critical path. If we give a range near critical path, DC will optimize all the path in the assigned range. If the range is large, DC will increase run time, so usually the range is defined at 10% of the clock period.

- Use the `-cricital_range` option of the `group_path` command
- Use the `set_critical_range` command

## Performing High-Effort Compile

High-Effort compile resynthesis on critical path, and doing restructure and remap on critical path.

Two commands:

- `compile_ultra` command. With 2 option which includes some script, providing additional timing and area optimization.
  - `-area_high_effort_script` option
  - `timing_high_effort_script` option

- `compile` command. with `map_effort -high` option

High-effort optimization of the critical path includes: Logic duplication and mapping to wide-fanin gates

- Use complex, high-fanin gate to replace some standard cell in critiical path can achieve better timing and area.
- For some high fanout or high load wire, DC will use logic duplication and restructuring to optimize the circuit. This may cause area increase.

## Performing High-Effort Compile Incremental Compile

Using incremental can improve circuit performance, if constraint is not meet after compile, maybe incremental can help.

Incremental only optimize gate-level, instead of the logic-level. The result maybe better or the same before optimization.

Incremental will use a lot of compute time, but it is the best way to reduce negative slack. To reduce compute time, we can set the module that meets timing requirement as dont_touch: `dont_touch noncritical_blocks`

Command: ```compile -map_effort high -incremental_mapping```

## Gate-Level Optimization

Gate-level optimization chooses proper standard cell in the library to optimize the design. It has 3 steps:

- delay optimization: optimize the delay, will also take DRC into consideration.
- design rule fixing: insert buffer, change cell size to meet DRC constraint. Usually will not affect timing and area but may cause optimization constraint violation.
- area recovery: optimize area, will not cause DRC and delay violation. Usually performed on non-critical path.

## Automatic ungrouping

Ungrouping removes hierarchy of the design, remove the hierarchy boundary, and allow DC ultra to reduce logic level to improve timing, and allow resource sharing to reduce area.

Two commands:

- `compile_ultra`: delay-based auto-ungrouping, area-based auto-ungrouping
- `compile -auto_ungroup`:

delay-based auto-ungrouping:

By default, compile_ultra will use this option. It will optimized around critical path. DesignWare component will not be optimized because DC treats them as already optimized structure.

area-based auto-ungrouping:

DC will estimize level or unmapping and remove some small sub-design to reduce area and increase timing.

## Adaptive retiming

Move register between timing path to improve performance. It may increase area or reduce area.

If we have critical path, retime will move register position. If not critical path, retime can reduce register number.

DC can only move registers with same timing constraint. DC will not move registers with different timing constraints.

After retiming, the cell name may have a R prefix and a serial number: `R_xxx`

Retime can't be used with the following option in compile_ultra: `-incremental`, `-top`, `-only_design_rule`

## High-level optimization and datapath optimization

DC use the follow method to optimize datapath:

- Use tree structure for arithmatic, for example, a + b + c + d will be optimized to (a + b) + (c + d)
- optimize logic function: a\*3\*5 will be optimized to a\*15
- Resource sharing

DC Ultra use the follow method to optimize datapath:

- Use designware library
- Reassign add/mult: (A\*C + B\*C) => (A+B)\*C
- Comparator sharing, A>B, A<B, and A<=B will use the same substractor.

## Verifying Functional Equivalence

The following optimization will cause mismatch between RTL design and netlist:

- register, port renaming caused by ungroup, group, uniquify, rename_design
- Equivalent register or inverted register being optimized, constant register being optimized
- Retiming cause circuit structure change
- Datapath optimization
- State machine optimization

DC will create setup files for formality (default file: default.svf)

## Partitioning for Synthesis

1. Do not let a comb logic go across multiple module.

DC will optimize the comb logic separately in each module which might increase area and timing.

2. Register output signal

Register all the output signal. Best structure and also simplify timing constraint.

Avoid glue logic in the top level partition. Move the glue logic into the sub module so top level partition only does port connection.

3. Choose design size based on synthesis time.

4. Separate logic part with other partition (PAD, PLL, etc.).