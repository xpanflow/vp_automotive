# Automotive virtual prototyping

This repository contains the electronic throttle control (ETC) benchmark from 
[Virtual Prototyping of Heterogeneous Automotive Applications: Matlab, SystemC, or both?](https://doi.org/10.1145/3287624.3287629).  
The case
study uses a Bosch DV-E5 throttle body to compare SystemC and MATLAB/Simulink
models driven by the same accelerator-pedal input.

SystemC uses transaction-level modeling (TLM) for the digital controller and
peripheral communication, and SystemC AMS for the mixed-signal interfaces and
electromechanical throttle body. The Simulink model uses Simscape for the
physical plant. The paper reports similar throttle responses in both models.
In the SystemC benchmark, dynamic time steps reduced simulation time, while
running compiled OpenRISC firmware in an instruction-set simulator increased it.


## Run the SystemC demo

On Linux, use CMake 3.13 or later in the 3.x series, a C++ compiler, and matching
SystemC 2.3.4 and SystemC AMS 2.3.4 libraries built with C++11, the standard
selected by this project. The commands below use the behavioral controller on a
setup without Or1ksim. [SystemC and SystemC AMS downloads](https://www.accellera.org/downloads/standards/systemc)

From the repository root, replace the two paths with your library installation
directories and run the following commands.

```sh
export SYSTEMC_BASE=/path/to/systemc
export SYSTEMCAMS_BASE=/path/to/systemc-ams

cmake -S vp_systemc/sc_models -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build --parallel
(cd vp_systemc/bin && ./scvp_etc_demo)
```

The demo simulates three seconds with dynamic timed data flow (DTDF) enabled.
It writes `vp_systemc/bin/tr_ecu_demo.vcd` with the pedal input, throttle
position, and motor current. Edit the pedal input or run length in
[`config_sim.h`](vp_systemc/sc_models/include/config_sim.h).

## Compare with MATLAB/Simulink

Use MATLAB with Simulink, Simscape, and Control System Toolbox. The models were
saved in R2017a.

1. In [`config_sim.h`](vp_systemc/sc_models/include/config_sim.h), set
   `k_trace_file` to `"csv"`. Rebuild and rerun the SystemC demo to generate
   the tabular trace at `vp_systemc/bin/tr_ecu_demo.dat`.
2. From the repository root in MATLAB, run the comparison script.

   ```matlab
   cd vp_matab
   load_sim_in_from_sc_sim
   ```

The script uses the SystemC pedal trace to drive `etc_demo_simc_performance`
and plots throttle position and motor current from both simulations.

## OpenRISC firmware

The C firmware and simulator configuration are in
[`vp_systemc/or1k_iss_firmware`](vp_systemc/or1k_iss_firmware) and
[`vp_systemc/or1ksim_cfg`](vp_systemc/or1ksim_cfg). The ISS variant requires
Or1ksim, an OpenRISC cross-compiler, and `ENABLE_ISS` in `config_sim.h`.
To run it, update the legacy source paths in the firmware CMake files and
set the cross-compiler path for your installation. Update the simulator
configuration and firmware image paths in the
[`sc_tlm_ecu_iss.cpp` wrapper](vp_systemc/sc_models/src/sc_tlm_ecu_iss.cpp).

## License

See the [GNU General Public License v3.0](LICENSE).
