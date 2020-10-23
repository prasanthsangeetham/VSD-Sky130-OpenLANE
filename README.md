# VSD-Sky130-OpenLANE
# **Overview**

OpenLANE is an automated RTL to GDSII flow based on several components including OpenROAD, Yosys, Magic, Netgen, Fault, OpenPhySyn, SPEF-Extractor and custom methodology scripts for design exploration and optimization. The flow performs full ASIC implementation steps from RTL all the way down to GDSII.

# **OpenLANE Architecture**
![](https://raw.githubusercontent.com/efabless/openlane/master/doc/openlane.flow.1.png)

# **OpenLANE Design Stages**
OpenLANE flow consists of several stages. By default all flow steps are run in sequence. Each stage may consist of multiple sub-stages. OpenLANE can also be run interactively as shown below.

You may run the flow interactively by using the `-interactive` option:

```
./flow.tcl -interactive
```

A tcl shell will be opened where the openlane package is automatically sourced:
```
% package require openlane 0.9
```

Then, you should be able to run the following commands:

0. Any tcl command.
1. `prep -design <design> -tag <tag> -config <config> -init_design_config -overwrite` similar to the command line arguments, design is required and the rest is optional
2. `run_synthesis` 
3. `run_floorplan`
4. `run_placement`
5. `run_cts`
6. `gen_pdn`
7. `run_routing`
8. `run_magic`
9. `run_magic_spice_export`
10. `run_magic_drc`
11. `run_lvs`
12. `run_magic_antenna_check`


The above commands can also be written in a file and passed to `flow.tcl`:

```
./flow.tcl -interactive -file <file>
```


**Note 1:** Currently, configuration variables have higher priority over the above commands so if `RUN_MAGIC` is 0, command `run_magic` will have no effect. 

**Note 2:** Currently, all these commands must be run in the flow sequence and no steps should be skipped.

**Note 3:** You can pass the -design, -tag, etc.. flags to ```./flow.tcl -interactive``` directly without the need of entering the interactive mode and then executing the prep command.

[0]:./OpenLANE_commands.md

**The following are the operations performed by different tools :**
1. **Synthesis**
    1. `yosys` - Performs RTL synthesis
    2. `abc` - Performs technology mapping
    3. `OpenSTA` - Pefroms static timing analysis on the resulting netlist to generate timing reports
2. **Floorplan and PDN**
    1. `init_fp` - Defines the core area for the macro as well as the rows (used for placement) and the tracks (used for routing)
    2. `ioplacer` - Places the macro input and output ports
    3. `pdn` - Generates the power distribution network
    4. `tapcell` - Inserts welltap and decap cells in the floorplan
3. **Placement**
    1. `RePLace` - Performs global placement
    2. `Resizer` - Performs optional optimizations on the design
    3. `OpenPhySyn` - Performs timing optimizations on the design
    4. `OpenDP` - Perfroms detailed placement to legalize the globally placed components
4. **CTS**
    1. `TritonCTS` - Synthesizes the clock distribution network (the clock tree)
5. **Routing** *
    1. `FastRoute` - Performs global routing to generate a guide file for the detailed router
    2. `TritonRoute` - Performs detailed routing
    3. `SPEF-Extractor` - Performs SPEF extraction
6. **GDSII Generation**
    1. `Magic` - Streams out the final GDSII layout file from the routed def
7. **Checks**
    1. `Magic` - Performs DRC Checks & Antenna Checks
    2. `Netgen` - Performs LVS Checks

**OpenLANE integrated several key open source tools over the execution stages:**
- RTL Synthesis, Technology Mapping, and Formal Verification : yosys + abc
- Static Timing Analysis: OpenSTA
- Floor Planning: init_fp, ioPlacer, pdn and tapcell
- Placement: RePLace (Global), Resizer and OpenPhySyn (Optimizations), and OpenDP (Detailed)
- Clock Tree Synthesis: TritonCTS
- Fill Insertion: OpenDP/filler_placement
- Routing: FastRoute (Global) and TritonRoute (Detailed)
- SPEF Extraction: SPEF-Extractor
- GDSII Streaming out: Magic
- DRC Checks: Magic
- LVS check: Netgen
- Antenna Checks: Magic

## OpenLANE Output

All output run data is placed by default under ./designs/design_name/runs. Each flow cycle will output timestamp-marked foler containing the following file structure:

```
designs/<design_name>
├── config.tcl
├── runs
│   ├── <tag>
│   │   ├── config.tcl
│   │   ├── logs
│   │   │   ├── cts
│   │   │   ├── floorplan
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   ├── reports
│   │   │   ├── cts
│   │   │   ├── floorplan
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   ├── results
│   │   │   ├── cts
│   │   │   ├── floorplan
│   │   │   ├── magic
│   │   │   ├── placement
│   │   │   ├── routing
│   │   │   └── synthesis
│   │   └── tmp
│   │       ├── cts
│   │       ├── floorplan
│   │       ├── magic
│   │       ├── placement
│   │       ├── routing
│   │       └── synthesis
```

To delete all generated runs under all designs:
- inside the docker:
    ```bash
        ./clean_runs.tcl
    ```
- outside the docker:
    ```bash
        make clean_runs
    ```

## Flow configuration

1. PDK / technology specific
2. Flow specific
3. Design specific

- A PDK should define at least one standard cell library(SCL) for the PDK. A common configuration file for all SCLs is located in:

    ```
    $PDK_ROOT/$PDK/config.tcl
    ```

    - Sometimes the PDK comes with several standard cell libraries. Each has an own configuration file that defines extra variables specific to the SCL. It may also override variables in the common PDK configuration file which is located in:

        ```
        $PDK_ROOT/$PDK/$STD_CELL_LIBRARY/config.tcl
        ```
    - More on configuring a new PDK in this [section](#setting-up-the-pdk-skywater-pdk)

- Flow specific variables are related to the flow and are initialized with default values in:

    ```
    ./configuration/
    ```

- Finally, each design should have it's own configuration file with some required variables which are available in this [list][17]. A design configuration file may override any of the variables defined in PDK or flow configuration files. This is the global configurations for the design:

    ```
    ./designs/<design>/config.tcl
    ```
