# Bandgap Reference Design

This project documents my work on the Bandgap Reference circuit using the Sky130 PDK.

## Objective
To design a stable voltage reference independent of temperature and process variations.

---

## 1. Introduction

A **Bandgap Reference (BGR)** circuit is an essential building block in analog and mixed-signal integrated circuits. Its main function is to generate a **stable reference voltage** that remains **independent of temperature, supply voltage, and process variations**. This stable voltage is used in many critical applications such as ADCs, DACs, voltage regulators, and biasing circuits.

<img width="698" height="268" alt="image" src="https://github.com/user-attachments/assets/d1801242-1362-4c9d-be72-87241838416d" />

The concept of the Bandgap Reference is based on combining two temperature-dependent voltages:

**CTAT (Complementary to Absolute Temperature) Voltage:**
* Typically derived from the base-emitter voltage (V<sub>BE</sub>) of a bipolar transistor.
* V<sub>BE</sub> decreases with increasing temperature (negative temperature coefficient).

<img width="825" height="295" alt="image" src="https://github.com/user-attachments/assets/8d27a9ba-6d6b-4963-82f8-c574f15db157" />

**PTAT (Proportional to Absolute Temperature) Voltage:**
* Generated from the difference in base-emitter voltages (ΔV<sub>BE</sub>) between two transistors operating at different current densities.
* ΔV<sub>BE</sub> increases with temperature (positive temperature coefficient).

<img width="328" height="627" alt="image" src="https://github.com/user-attachments/assets/ee122f26-e0ee-4478-83bf-1fd8fb7954ac" />

By carefully scaling and summing these two voltages, the opposing temperature effects cancel out, resulting in a **constant output voltage**—typically around **1.2 V**, which corresponds to the bandgap voltage of silicon at 0 K.

### 1.1 Working Principle (Simplified)
At its core, the Bandgap Reference circuit works as follows:
1. Generate a CTAT voltage from a diode-connected BJT (V<sub>BE</sub>).
2. Generate a PTAT voltage using two BJTs with different emitter area ratios.
3. Add the PTAT voltage (positive TC) to the CTAT voltage (negative TC) in proper proportions.
4. The resulting sum is a temperature-independent reference voltage.

### 1.2 Features of Bandgap Reference (BGR)
* Temperature-independent voltage reference circuit widely used in Integrated Circuits (ICs).
* Produces a constant output voltage regardless of power supply variation, temperature changes, and circuit loading.
* Typical output voltage ≈ **1.2 V**, which is close to the bandgap energy of silicon at 0 K.
* Used in almost all types of circuits — analog, digital, mixed-signal, RF, and System-on-Chip (SoC) designs.

<img width="937" height="571" alt="image" src="https://github.com/user-attachments/assets/834f328b-4c6b-47ce-8d3a-04dcef6688b4" />

---

## 2. Tool and PDK Setup

### 2.1 Tools Setup
For the design, simulation, and verification of the Bandgap Reference (BGR) circuit, the following open-source EDA tools are used:

**Ngspice — Circuit Simulation**
Ngspice is an open-source SPICE-based simulator used for performing analog circuit simulations. It takes a SPICE netlist as input, which describes the circuit components and their connections, and then computes electrical parameters such as node voltages, currents, and transfer characteristics. 

In this project, Ngspice is used to simulate the schematic-level design of the BGR circuit, analyze DC, AC, and transient behavior, and verify temperature dependence and output voltage stability.

Magic — Layout Design and DRC
Magic is a VLSI layout editor developed by Berkeley, primarily used for IC layout design in open-source PDKs such as Sky130. It provides interactive tools for drawing transistors, interconnects, and layers according to process design rules.

Magic is used here to create the physical layout of the BGR circuit, perform Design Rule Checks (DRC), and extract the layout to generate a SPICE netlist for post-layout simulations.

wget [http://opencircuitdesign.com/magic/archive/magic-8.3.32.tgz](http://opencircuitdesign.com/magic/archive/magic-8.3.32.tgz)
tar xvfz magic-8.3.32.tgz
cd magic-8.3.32
./configure
sudo make
sudo make install

Netgen — LVS (Layout vs. Schematic)
Netgen is a layout verification tool used for Layout Versus Schematic (LVS) comparison. It compares the netlist extracted from the layout (using Magic) with the schematic netlist (used in Ngspice simulation) to verify connectivity and device matching. A successful LVS ensures that the layout accurately represents the schematic, confirming the design's electrical integrity before fabrication.

git clone git://[opencircuitdesign.com/netgen](https://opencircuitdesign.com/netgen)
cd netgen
./configure
sudo make
sudo make install

2.2 PDK Setup
The SkyWater sky130 PDK provides process design data (layers, device rules, models) required for layout, extraction, and simulation.

Steps for setup:

Create a directory for the PDK.

Clone the SkyWater PDK repository.

Initialize submodules (if required).

Build or install PDK libraries (optional).

Set the PDK path so tools like Magic, Ngspice, and Netgen can locate it easily.

<img width="953" height="1006" alt="image" src="https://github.com/user-attachments/assets/12bc21fa-9c67-4edc-b1e5-222b2808dd9a" />
<img width="964" height="1015" alt="image" src="https://github.com/user-attachments/assets/c49a6a82-d6ac-4e48-bc4d-d1a4f5e23004" />
<img width="958" height="984" alt="image" src="https://github.com/user-attachments/assets/d86e1249-b3a3-4144-b85a-4997d9ff3d41" />



3. The BGR Principle
3.1 Core Concepts
The primary goal of a Bandgap Reference (BGR) circuit is to generate a robust reference voltage that remains completely independent of temperature fluctuations. It achieves this by combining two voltage components that react to temperature in perfectly opposite ways:

CTAT (Complementary to Absolute Temperature): A voltage that decreases as temperature increases. In silicon, the base-emitter voltage (V<sub>BE</sub>) of a forward-biased Bipolar Junction Transistor (BJT) or diode naturally exhibits this behavior, dropping at a rate of approximately -2 mV/°C.

PTAT (Proportional to Absolute Temperature): A voltage that increases as temperature increases. We can generate this by extracting the difference between the V<sub>BE</sub> of two transistors (ΔV<sub>BE</sub>) that are operating at different current densities.

⚙️ Principle of Operation
By carefully extracting the PTAT component and adding it to the CTAT component in a specific ratio, the upward and downward thermal slopes cancel each other out. The resulting sum is a constant, temperature-independent reference voltage of approximately 1.2 V—which corresponds to the bandgap energy of silicon at 0 K.

3.2 Types of Bandgap References
BGR circuits can be categorized based on their underlying architecture and their specific application targets.

🧩 Architecture-wise Classification

Self-Biased Current Mirror: Utilizes internal transistor feedback for biasing. It is highly favored for its simplicity, stability, and ease of integration into mixed-signal ICs.

Operational Amplifier-Based: Uses an op-amp to force precise node voltages. While it offers superior accuracy and matching, it consumes more power and area.

⚙️ Application-wise Classification

Low-Voltage BGR: Designed to operate under constrained supply voltages.

Low-Power BGR: Optimized for battery-powered or energy-harvesting systems.

High-PSRR / Low-Noise BGR: Engineered to reject power supply ripples and minimize internal noise.

🧠 Our Design Choice
For this project, I have implemented a Self-Biased Current Mirror Architecture. This topology strikes an excellent balance between design simplicity, power efficiency, and reliable temperature stability without the overhead of an external operational amplifier.

3.3 Self-Biased Current Mirror Based BGR
This specific BGR architecture is constructed from several specialized sub-blocks that work together to maintain equilibrium and generate the 1.2 V reference.

🧩 Core Components:

CTAT Generator: Produces the downward-sloping thermal voltage (using a diode-connected BJT).

PTAT Generator: Produces the upward-sloping thermal voltage (using an array of matched BJTs and resistors).

Self-Biased Current Mirror: A specialized current mirror that requires no external biasing. It uses internal feedback loops to automatically establish and stabilize its own operating current.

Reference Branch: The core summing node. A mirror transistor drives the PTAT current into a branch containing a resistor and a CTAT diode. The voltage drop across the resistor scales the PTAT slope to match the CTAT slope. The total voltage across this branch is our stable 1.2 V reference.

Start-up Circuit: A critical safety mechanism. Self-biased circuits inherently possess a "zero-current" degenerative state. The start-up circuit detects this dead state and injects a small initial current to kickstart the mirror, safely turning off once equilibrium is reached.

Advantages of this Topology:

Simplicity: A straightforward, amplifier-free structure that is easy to design and layout.

Inherent Stability: The internal self-biasing mechanism ensures a reliable operating point once active.

Limitations to Consider:

Lower PSRR: More susceptible to variations in the main power supply (cascoding is often required to fix this).

Voltage Headroom: Stacked transistors limit how low the main supply voltage can go.

Start-up Dependency: Absolutely requires a robust start-up circuit to prevent failure at power-on.

4. Design and Pre-layout Simulation
For the practical implementation of the Bandgap Reference (BGR) circuit, the SkyWater SKY130 (130 nm) PDK is used. Before designing the complete circuit, the specific design requirements must be defined.

<img width="260" height="182" alt="{769D0A2F-B68E-4204-8F2F-B5A4FA718775}" src="https://github.com/user-attachments/assets/57964431-6dba-4656-8156-bd216c2c1ed8" />
<img width="372" height="307" alt="{779F36D1-4EA9-46B0-95E2-8B12FA69E2CE}" src="https://github.com/user-attachments/assets/4938fc80-78bc-4750-a14b-430a5679c6db" />
<img width="249" height="135" alt="{1FA19F90-2A9A-46EC-A2B4-3373E662BDCC}" src="https://github.com/user-attachments/assets/689144aa-912d-470e-ac94-6803ca18678b" />

4.1 Current Calculation
Maximum Power Consumption: 60 µW

Supply Voltage: 1.8 V

Total Current: 60 µW / 1.8 V = 33.33 µA

Branch Current: 10 µA per branch is selected (3 branches × 10 µA = 30 µA).

Start-up current: 1–2 µA.

4.2 Choosing Number of BJTs in Branch 2
Fewer BJTs lead to smaller resistance but poorer matching, while more BJTs lead to higher resistance but better matching.

Chosen compromise: 8 BJTs in parallel for good matching and moderate resistance.

4.3 Calculation of R1
Formula: R1 = (Vt × ln(8)) / I

Calculation: R1 = (26 mV × ln(8)) / 10.7 µA ≈ 5 kΩ

R1 Size: W = 1.41 µm, L = 7.8 µm

Unit resistance: 2 kΩ

Resistor implementation: 2 in series and 2 in parallel (2 + 2 + (2‖2))

4.4 Calculation of R2
Current through reference branch: I3 = I2 = (Vt × ln(8)) / R1

Voltage across R2: VR2 = R2 × I3 = (R2 / R1) × (Vt × ln(8))

Slope of VR2: (R2 / R1) × (ln(8) × 115 µV/°C)

Slope of VQ3: -1.6 mV/°C

For a zero temperature coefficient, the total slope must equal 0.

Calculated R2: ≈ 33 kΩ

Resistor implementation: 16 in series and 2 in parallel (2 + 2 + … + 2 + (2‖2))

4.5 Self-Biased Current Mirror (SBCM) Design
A. PMOS Design (MP1, MP2)

Operate both transistors in the saturation region.

Increase channel length to reduce channel length modulation.

Final size: L = 2 µm, W = 5 µm, M = 4

B. NMOS Design (MN1, MN2)

Operate both transistors either in the saturation or deep subthreshold region. Here, they are designed to work in the deep subthreshold region.

Increase channel length to improve stability.

Final size: L = 1 µm, W = 5 µm, M = 8

<img width="1027" height="647" alt="image" src="https://github.com/user-attachments/assets/03bede6e-4f9e-468a-8451-ead60f10eaf3" />

## 5. CTAT Voltage Generation and Simulation

The first practical step in building the BGR is designing and verifying the **CTAT (Complementary to Absolute Temperature)** block. This block utilizes a diode-connected PNP Bipolar Junction Transistor (BJT) to generate a voltage that decreases linearly as temperature increases.

### 5.1 CTAT Circuit Netlist (`ctat_voltage_gen.sp`)
To generate the CTAT voltage, a single BJT (`sky130_fd_pr__pnp_05v5_W3p40L3p40`) is connected in a diode configuration (base and collector tied to ground). An ideal DC current source of `10uA` is fed into the emitter. 

A DC temperature sweep (`.dc temp -40 125 5`) is defined to simulate the voltage response across the BJT from -40°C to 125°C.

![CTAT Netlist] ### 5.2 Troubleshooting Sky130 PDK Errors
During the initial simulation attempt using Ngspice-45+, a fatal error occurred preventing the simulation from running.

**The Error:**
The simulator crashed with: `Error: Could not find include file ../../../cells/rf_nfet_01v8/sky130_fd_pr__rf_nfet_01v8_b__tt.corner.spice`.
This happens because the open-source Sky130 library sometimes contains broken relative paths to RF components that are not required for this basic analog design.
<img width="1024" height="674" alt="image" src="https://github.com/user-attachments/assets/9d9526d9-2e86-49f3-ad46-2deb48c5e7a6" />


**The Fix:**
Instead of manually hunting down the broken file, I used a Linux stream editor (`sed`) command to automatically find the broken `.include` line inside the `tt` corner file and comment it out (adding an asterisk `*` at the beginning of the line).

bash
sed -i '/\.include/s/^/* /' /home/vsduser/cad_vsd/eda-technology/sky130/models/spice/models/corners/tt/rf.spice.

After patching the library file, the simulation compiled and ran successfully.![PDK Error and Sed Fix] ### 5.3 CTAT Simulation ResultsPlotting the emitter voltage v(qp1) against the temperature sweep yields the expected CTAT behavior.As seen in the graph below, the voltage starts high at lower temperatures (around 850 mV at -40°C) and drops linearly as the temperature increases to 125°C.![CTAT Voltage Graph] ### 5.4 Temperature Coefficient (Slope) VerificationTo verify the exact temperature coefficient, we measured the slope of the V<sub>BE</sub> curve ($dy/dx$). I also expanded the simulation to test a multiple-BJT configuration (ctat_voltage_gen_mult_bjt.sp) to observe how parallel devices behave.Using Ngspice's interactive measurement tools, the slope of the curve was calculated as:dy/dx = -0.00173136 V/°C * Temperature Coefficient ≈ -1.73 mV/°CThis perfectly aligns with the theoretical expectation of approximately -2 mV/°C, confirming that the CTAT block is functioning correctly and is ready to be integrated into the main Bandgap Reference circuit.
<img width="1024" height="542" alt="image" src="https://github.com/user-attachments/assets/ebfbafb0-f600-4272-a19c-fb963a488b19" />

### 5.5 CTAT Voltage under Variable Bias Current:
To further understand the behavior of the CTAT block, it is important to observe how the base-emitter voltage ($V_{BE}$) changes under different biasing conditions. The $V_{BE}$ of a BJT is logarithmically dependent on its collector/emitter current.What We Did:I created a copy of the original CTAT netlist (ctat_voltage_gen_var_current.sp) and modified the .dc control statement to perform a nested parametric sweep. Instead of simulating just one fixed 10 µA current, the simulator sweeps the temperature from -40°C to +125°C while simultaneously stepping the bias current through multiple values.Simulation Results & Graph Analysis:Plotting the emitter voltage v(qp1) under these conditions yields a "family of curves."Each individual red line represents the CTAT voltage drop across the BJT at a specific, fixed bias current.Vertical Shift: As the bias current increases, the entire CTAT line shifts upward (resulting in a higher overall $V_{BE}$).Slope Variation: The terminal output shows slope calculations ($dy/dx$) for different curves (e.g., -1.92 mV/°C, -1.74 mV/°C). This demonstrates that changing the bias current slightly alters the Temperature Coefficient (TC) of the diode.
<img width="1024" height="432" alt="image" src="https://github.com/user-attachments/assets/1bdb4a2b-f48e-4335-b8ca-1e7e7a29447d" />

## 6. PTAT Voltage Generation and Simulation

With the CTAT block verified, the next step is generating the **PTAT (Proportional to Absolute Temperature)** voltage. This voltage must increase with temperature to successfully cancel out the CTAT's negative slope. 

A PTAT voltage is generated by extracting the difference in base-emitter voltages ($\Delta V_{BE}$) between two BJTs operating at different current densities. To achieve this, the circuit uses one BJT in the first branch and eight identical BJTs connected in parallel in the second branch.

### 6.1 Initial Simulation and Ammeter Errors
During the initial attempt to measure the branch currents, Ngspice threw a `no such vector vid1#branch` fatal error. 

**The Fix:**
In SPICE simulators, measuring current directly through a wire is not natively supported. To plot current, you must insert a "dummy" 0-volt DC voltage source in series with the branch to act as a physical ammeter. I modified the netlist to include `vid1` and `vid2` (0V sources) to successfully extract the current data.

![Ammeter Error] ### 6.2 The Flatline Current Issue
After fixing the ammeter error, plotting the current resulted in a perfectly flat horizontal line at 10 µA across all temperatures. 

**The Cause:** The preliminary netlist used ideal, fixed DC current sources (`isup1 node1 qp1 dc 10u`). An ideal source will force exactly 10 µA regardless of temperature, which prevents the natural PTAT slope from forming. 

![Flatline Current] ### 6.3 Behavioral Source Implementation & Debugging
To generate a true PTAT current without prematurely introducing the complex PMOS current mirror, I replaced the ideal current sources with **Behavioral Current Sources (B-sources)**. This allows the simulator to dynamically calculate the current based on the voltage difference between the two BJT branches.

**The Syntax Bug:**
My initial B-source equation was: I=(v(qp1)-v(qp2))/4890 + 0.1u 
However, the simulator produced a flat graph at near absolute zero (measured in femtoamps `fA`). This occurred because the strict mathematical engine inside Ngspice's B-source does not recognize the `u` (micro) suffix, treating the startup current as zero and preventing the BJTs from turning on.

![fA Crash Graph] **The Fix:**
I corrected the syntax using strict scientific notation, providing a 1 nanoamp "spark" to initiate the self-biasing loop:
spice
B1 node1 qp1 I='(v(qp1)-v(qp2))/4890 + 1e-9'
B2 node2 qp2 I='(v(qp1)-v(qp2))/4890 + 1e-9'

## 6.4 Final PTAT Current Verification:
With the behavioral sources correctly implemented, the simulation successfully generated the PTAT current. As shown below, the current through both branches perfectly overlaps and slopes upwards linearly as temperature increases, crossing ~10 µA at 0°C.![PTAT Current Graph] ### 6.5 Final PTAT Voltage VerificationThe ultimate goal of this block is the $\Delta V_{BE}$ voltage. To verify this, I plotted three vectors:v(ra1): The voltage across the single BJT branch (Blue line).v(qp2): The voltage across the 8-parallel BJT branch (Red line).v(ra1) - v(qp2): The difference between them (Yellow line).The resulting graph perfectly demonstrates the PTAT principle. While the individual BJT voltages drop with temperature (CTAT behavior), the difference between them (the yellow line) starts low and slopes upwards, providing the exact positive temperature coefficient needed for the final Bandgap Reference.
<img width="1024" height="528" alt="image" src="https://github.com/user-attachments/assets/6296bd2b-286e-4bc6-bbb3-0f3b95121e7c" />
<img width="1600" height="754" alt="image" src="https://github.com/user-attachments/assets/c9045d33-c662-428f-b7d1-2f99d33de66a" />

## 7. Ideal Op-Amp Based Bandgap Reference Simulation

After successfully isolating and verifying both the CTAT and PTAT behaviors, the next step is to combine them into a complete Bandgap Reference (BGR) circuit. Before implementing a complex, transistor-level operational amplifier, it is standard practice to first simulate the BGR core using an **Ideal Op-Amp** (modeled as a Voltage-Controlled Voltage Source, or VCVS).

### 7.1 Circuit Architecture
The circuit uses a VCVS with a high gain (1000) to enforce equal voltages at the top of the CTAT and PTAT branches. PMOS transistors (`xmp1`, `xmp2`, `xmp3`) act as a current mirror, ensuring identical current flows through all three branches. 

1. **Branch 1 (CTAT):** A single BJT (`Q1`).
2. **Branch 2 (PTAT):** A resistor (`R1`) and 8 parallel BJTs (`Q2`).
3. **Branch 3 (Reference):** A scaling resistor (`R2`) and a single BJT (`Q3`). The voltage at the top of this branch is our final $V_{ref}$.

![BGR Ideal Op-Amp Schematic] ![BGR Ideal Op-Amp Netlist] ### 7.2 Resolving Ngspice-45+ Compatibility and Convergence Issues
During the initial simulation runs, two major SPICE errors occurred which required netlist modifications to ensure successful compilation.

**1. The "Too Few Parameters" Error:**
The simulation halted immediately with a parameter error targeting the `xqp` (BJT) instances. 
* **Cause:** Newer, stricter versions of Ngspice (v45+) reject the `m=` (multiplier) shortcut for the Sky130 BJT subcircuits, as the model explicitly expects a fixed number of terminal nodes. 
* **Solution:** To maintain exact electrical equivalence with the instructor's design, I replaced the `m=8` parameter by explicitly instantiating 8 individual Sky130 BJT models connected in parallel.

![BJT Parameter Error] **2. The "Singular Matrix" Convergence Error:**
After fixing the BJT parameters, the simulator threw a `Singular matrix: check node vref` error and failed to compute the DC sweep.
* **Cause:** An ideal VCVS has infinite bandwidth and no supply limits. When a DC sweep begins at extremely low temperatures (e.g., -40°C or 0°C), initial node voltages are zero. The ideal op-amp responds with an infinite impulse, causing the mathematical matrix to become singular (divide-by-zero) and crash. Furthermore, unassigned dummy nodes (like a typo of `ref` instead of `vref` in the plot command) can leave nodes floating.
* **Solution:** I ensured all dummy ammeters (`vid1`, `vid2`, `vid3`) were correctly wired to provide a path to ground. To assist the SPICE convergence engine, the DC sweep was adjusted to begin at a stable room temperature (`.dc temp 27 125 5`), allowing the matrix to calculate the initial steady-state operating point successfully.

### 7.3 Internal Node Verification
Before checking the final output, I verified that the internal feedback loops were operating correctly.

**Op-Amp Voltage Tracking:**
By plotting `v(qp1)` and `v(ra1)`, we can confirm that the VCVS is successfully forcing the two branch voltages to be identical. The graph shows the two lines perfectly overlapping, proving the op-amp is maintaining the virtual short.
<img width="584" height="510" alt="image" src="https://github.com/user-attachments/assets/d43f3498-ae51-4a6d-9e09-cf759e310a22" />
<img width="1087" height="793" alt="image" src="https://github.com/user-attachments/assets/ab18eb5e-3543-4cc3-97a1-a2752b566d54" />

![Op-Amp Tracking Graph] **PTAT Current Generation:**
By plotting the current through the dummy ammeters (`vid1#branch` and `vid2#branch`), the graph confirms that the PMOS current mirrors and the BJT $\Delta V_{BE}$ are successfully generating a PTAT current that slopes linearly upwards.
<img width="1600" height="899" alt="image" src="https://github.com/user-attachments/assets/75f3e8f0-300e-41dc-a871-fbe19c1bc7b3" />

### 7.4 Final Bandgap Voltage Verification
With the internal mechanics verified, I plotted the final reference voltage. 

**The BGR Parabola:**
Plotting `v(vref)` yields the classic "Bandgap Parabola." Because the CTAT and PTAT slopes are not perfectly linear across all extremes of temperature, their sum creates a slight curvature. The voltage remains incredibly stable, peaking precisely around **1.237 V**, confirming a highly stable reference.
<img width="1024" height="567" alt="image" src="https://github.com/user-attachments/assets/226b328d-0650-4828-b6f9-745d5b64fa23" />

![BGR Vref Parabola] **The Component Breakdown (The "Holy Grail" Plot):**
To visually demonstrate the core principle of the Bandgap Reference, I plotted the constituent voltages on a single graph:
* **Blue Line `v(qp3)`:** The CTAT voltage component, decreasing with temperature.
* **Red Line `v(vref) - v(qp3)`:** The scaled PTAT voltage across the resistor, increasing with temperature.
* **Orange Line `v(vref)`:** The sum of the Red and Blue lines. The opposing slopes perfectly cancel out, resulting in the flat, stable 1.23 V reference.
<img width="553" height="436" alt="image" src="https://github.com/user-attachments/assets/a70c2f50-97f0-418f-a004-22b91bc835c6" />

![BGR Component Breakdown] This successfully completes the verification of the Ideal Op-Amp BGR topology. The next stage involves replacing the ideal VCVS with a real, transistor-level Self-Biased Current Mirror.

