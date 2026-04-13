# Bandgap Reference Design

This project documents my work on the Bandgap Reference circuit using the Sky130 PDK.

## Objective
To design a stable voltage reference independent of temperature and process variations.

## INTRODUCTION

A **Bandgap Reference (BGR)** circuit is an essential building block in analog and mixed-signal integrated circuits. Its main function is to generate a **stable reference voltage** that remains **independent of temperature, supply voltage, and process variations**. This stable voltage is used in many critical applications such as ADCs, DACs, voltage regulators, and biasing circuits.
<img width="698" height="268" alt="image" src="https://github.com/user-attachments/assets/d1801242-1362-4c9d-be72-87241838416d" />


The concept of the Bandgap Reference is based on combining two temperature-dependent voltages:

1. **CTAT (Complementary to Absolute Temperature) Voltage:**
   * Typically derived from the base-emitter voltage (V<sub>BE</sub>) of a bipolar transistor.
   * V<sub>BE</sub> decreases with increasing temperature (negative temperature coefficient).
   * <img width="825" height="295" alt="image" src="https://github.com/user-attachments/assets/8d27a9ba-6d6b-4963-82f8-c574f15db157" />


2. **PTAT (Proportional to Absolute Temperature) Voltage:**
   * Generated from the difference in base-emitter voltages (ΔV<sub>BE</sub>) between two transistors operating at different current densities.
   * ΔV<sub>BE</sub> increases with temperature (positive temperature coefficient).
   * <img width="328" height="627" alt="image" src="https://github.com/user-attachments/assets/ee122f26-e0ee-4478-83bf-1fd8fb7954ac" />


By carefully scaling and summing these two voltages, the opposing temperature effects cancel out, resulting in a **constant output voltage**—typically around **1.2 V**, which corresponds to the bandgap voltage of silicon at 0 K.

### Working Principle (Simplified)
At its core, the Bandgap Reference circuit works as follows:
1. Generate a CTAT voltage from a diode-connected BJT (V<sub>BE</sub>).
2. Generate a PTAT voltage using two BJTs with different emitter area ratios.
3. Add the PTAT voltage (positive TC) to the CTAT voltage (negative TC) in proper proportions.
4. The resulting sum is a temperature-independent reference voltage.

### Features of Bandgap Reference (BGR)
* Temperature-independent voltage reference circuit widely used in Integrated Circuits (ICs).
* Produces a constant output voltage regardless of power supply variation, temperature changes, and circuit loading.
* Typical output voltage ≈ **1.2 V**, which is close to the bandgap energy of silicon at 0 K.
* Used in almost all types of circuits — analog, digital, mixed-signal, RF, and System-on-Chip (SoC) designs.
  <img width="937" height="571" alt="image" src="https://github.com/user-attachments/assets/834f328b-4c6b-47ce-8d3a-04dcef6688b4" />


---

## 1. Tool and PDK Setup

### 1.1 Tools Setup
For the design, simulation, and verification of the Bandgap Reference (BGR) circuit, the following open-source EDA tools are used:

#### Ngspice — Circuit Simulation
**Ngspice** is an open-source SPICE-based simulator used for performing analog circuit simulations. It takes a SPICE netlist as input, which describes the circuit components and their connections, and then computes electrical parameters such as node voltages, currents, and transfer characteristics.
In this project, Ngspice is used to:
* Simulate the schematic-level design of the BGR circuit.
* Analyze DC, AC, and transient behavior.
* Verify temperature dependence and output voltage stability.

**Steps to install Ngspice:**
bash    
sudo apt-get install ngspice
Magic — Layout Design and DRC
Magic is a VLSI layout editor developed by Berkeley, primarily used for IC layout design in open-source PDKs such as Sky130. It provides interactive tools for drawing transistors, interconnects, and layers according to process design rules.
Magic is used here to:

Create the physical layout of the BGR circuit.

Perform Design Rule Check (DRC) to ensure the layout complies with the fabrication constraints of the Sky130 process.

Extract the layout to generate a SPICE netlist for post-layout simulations.
wget [http://opencircuitdesign.com/magic/archive/magic-8.3.32.tgz](http://opencircuitdesign.com/magic/archive/magic-8.3.32.tgz)
tar xvfz magic-8.3.32.tgz
cd magic-8.3.32
./configure
sudo make
sudo make install

Netgen — LVS (Layout vs. Schematic)
Netgen is a layout verification tool used for Layout Versus Schematic (LVS) comparison. It compares the netlist extracted from the layout (using Magic) with the schematic netlist (used in Ngspice simulation) to verify connectivity and device matching.
A successful LVS ensures that the layout accurately represents the schematic, confirming the design's electrical integrity before fabrication.
git clone git://[opencircuitdesign.com/netgen](https://opencircuitdesign.com/netgen)
cd netgen
./configure
sudo make
sudo make install

### 1.2 PDK Setup
The SkyWater sky130 PDK provides process design data (layers, device rules, models) required for layout, extraction and simulation. Below are typical steps to obtain and prepare the SkyWater-130 PDK on a Linux development environment.
Steps:

Create a directory for the PDK.

Clone the SkyWater PDK repository.

Initialize submodules (if required).

Build or install PDK libraries (optional).

Set the PDK path so tools like Magic, Ngspice, and Netgen can locate it easily.

<img width="953" height="1006" alt="image" src="https://github.com/user-attachments/assets/12bc21fa-9c67-4edc-b1e5-222b2808dd9a" />
<img width="964" height="1015" alt="image" src="https://github.com/user-attachments/assets/c49a6a82-d6ac-4e48-bc4d-d1a4f5e23004" />

## 2. Bandgap Reference (BGR) Introduction

### 2.1 The BGR Principle
The primary goal of a Bandgap Reference (BGR) circuit is to generate a robust reference voltage that remains completely independent of temperature fluctuations. It achieves this by combining two voltage components that react to temperature in perfectly opposite ways:

* **CTAT (Complementary to Absolute Temperature):** A voltage that *decreases* as temperature increases. In silicon, the base-emitter voltage (V<sub>BE</sub>) of a forward-biased Bipolar Junction Transistor (BJT) or diode naturally exhibits this behavior, dropping at a rate of approximately -2 mV/°C.
* **PTAT (Proportional to Absolute Temperature):** A voltage that *increases* as temperature increases. We can generate this by extracting the difference between the V<sub>BE</sub> of two transistors (ΔV<sub>BE</sub>) that are operating at different current densities.

**⚙️ Principle of Operation**
By carefully extracting the PTAT component and adding it to the CTAT component in a specific ratio, the upward and downward thermal slopes cancel each other out. The resulting sum is a constant, temperature-independent reference voltage of approximately **1.2 V**—which corresponds to the bandgap energy of silicon at 0 K.

---

### 2.2 Types of Bandgap References
BGR circuits can be categorized based on their underlying architecture and their specific application targets:

**🧩 Architecture-wise Classification**
1. **Self-Biased Current Mirror:** Utilizes internal transistor feedback for biasing. It is highly favored for its simplicity, stability, and ease of integration into mixed-signal ICs.
2. **Operational Amplifier-Based:** Uses an op-amp to force precise node voltages. While it offers superior accuracy and matching, it consumes more power and area.

**⚙️ Application-wise Classification**
* **Low-Voltage BGR:** Designed to operate under constrained supply voltages.
* **Low-Power BGR:** Optimized for battery-powered or energy-harvesting systems.
* **High-PSRR / Low-Noise BGR:** Engineered to reject power supply ripples and minimize internal noise.

**🧠 Our Design Choice**
For this project, I have implemented a **Self-Biased Current Mirror Architecture**. This topology strikes an excellent balance between design simplicity, power efficiency, and reliable temperature stability without the overhead of an external operational amplifier.

---

### 2.3 Self-Biased Current Mirror Based BGR
This specific BGR architecture is constructed from several specialized sub-blocks that work together to maintain equilibrium and generate the 1.2V reference.

**🧩 Core Components:**
1. **CTAT Generator:** Produces the downward-sloping thermal voltage (using a diode-connected BJT).
2. **PTAT Generator:** Produces the upward-sloping thermal voltage (using an array of matched BJTs and resistors).
3. **Self-Biased Current Mirror:** A specialized current mirror that requires no external biasing. It uses internal feedback loops to automatically establish and stabilize its own operating current. This makes the circuit highly compact and self-sufficient.
4. **Reference Branch:** The core summing node. A mirror transistor drives the PTAT current into a branch containing a resistor and a CTAT diode. By sizing the resistor appropriately, the voltage drop across the resistor scales the PTAT slope to perfectly match the CTAT slope. The total voltage across this branch is our stable 1.2V reference.
5. **Start-up Circuit:** A critical safety mechanism. Self-biased circuits inherently possess a "zero-current" degenerative state where the circuit remains off indefinitely. The start-up circuit detects this dead state and injects a small initial current to kickstart the mirror. Once the main circuit reaches its self-biasing equilibrium, the start-up circuit safely turns off.

**Advantages of this Topology**
* **Simplicity:** A straightforward, amplifier-free structure that is easy to design and layout.
* **Inherent Stability:** The internal self-biasing mechanism ensures a reliable operating point once active.

**Limitations to Consider**
* **Lower PSRR:** Without an op-amp, the circuit is more susceptible to variations in the main power supply (cascoding is often required to fix this).
* **Voltage Headroom:** The stacked transistors limit how low the main supply voltage can go.
* **Start-up Dependency:** It absolutely requires a robust start-up circuit to prevent failure at power-on.

### 3. Design and Pre-layout Simulation
For the practical implementation of the Bandgap Reference (BGR) circuit, the SkyWater SKY130 (130 nm) PDK is used.
Before designing the complete circuit, we must first define the design requirements that our circuit should meet.
<img width="260" height="182" alt="{769D0A2F-B68E-4204-8F2F-B5A4FA718775}" src="https://github.com/user-attachments/assets/57964431-6dba-4656-8156-bd216c2c1ed8" />
<img width="372" height="307" alt="{779F36D1-4EA9-46B0-95E2-8B12FA69E2CE}" src="https://github.com/user-attachments/assets/4938fc80-78bc-4750-a14b-430a5679c6db" />
<img width="249" height="135" alt="{1FA19F90-2A9A-46EC-A2B4-3373E662BDCC}" src="https://github.com/user-attachments/assets/689144aa-912d-470e-ac94-6803ca18678b" />

### 3.3 Circuit Design
1. Current Calculation
Maximum Power Consumption = 60 µW
Supply Voltage = 1.8 V

Total Current = 60 µW / 1.8 V = 33.33 µA

Hence, 10 µA per branch is selected (3 × 10 = 30 µA)
Start-up current = 1–2 µA

2. Choosing Number of BJTs in Branch 2
  Fewer BJTs → smaller resistance but poorer matching
  More BJTs → higher resistance but better matching
  Chosen compromise: 8 BJTs in parallel for good matching and moderate resistance.

3. Calculation of R1
  R1 = (Vt × ln(8)) / I
  R1 = (26 mV × ln(8)) / 10.7 µA ≈ 5 kΩ

R1 Size:
  W = 1.41 µm
  L = 7.8 µm
  Unit resistance = 2 kΩ
  Resistor implementation: 2 in series and 2 in parallel (2 + 2 + (2‖2))

4. Calculation of R2
  Current through reference branch:
  I3 = I2 = (Vt × ln(8)) / R1
  Voltage across R2:
  VR2 = R2 × I3 = (R2 / R1) × (Vt × ln(8))
  Slope of VR2 = (R2 / R1) × (ln(8) × 115 µV/°C)
  Slope of VQ3 = -1.6 mV/°C

  For zero temperature coefficient,
  Total slope = 0 → R2 ≈ 33 kΩ
  Resistor implementation: 16 in series and 2 in parallel (2 + 2 + … + 2 + (2‖2))

5. Self-Biased Current Mirror (SBCM) Design
    A. PMOS Design (MP1, MP2)
     Operate both transistors in saturation region.
     Increase channel length to reduce channel length modulation.
     Final size: L = 2 µm, W = 5 µm, M = 4

    B. NMOS Design (MN1, MN2)
     Operate both transistors either in saturation or deep subthreshold region.
     Here, they are designed to work in deep subthreshold region.
     Increase channel length to improve stability.
     Final size: L = 1 µm, W = 5 µm, M = 8
<img width="1027" height="647" alt="image" src="https://github.com/user-attachments/assets/03bede6e-4f9e-468a-8451-ead60f10eaf3" />





