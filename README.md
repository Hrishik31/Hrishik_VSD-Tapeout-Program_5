# OpenROAD Installation and Floorplanning Guide

## 📚 Table of Contents
- [Overview](#overview)
- [Why This Task is Important](#why-this-task-is-important)
- [Prerequisites](#prerequisites)
- [Installation Steps](#installation-steps)
  - [Step 1: Clone OpenROAD Flow Scripts](#step-1-clone-openroad-flow-scripts)
  - [Step 2: Build and Install OpenROAD](#step-2-build-and-install-openroad)
  - [Step 3: Verify Installation](#step-3-verify-installation)
- [Running Floorplan and Placement](#running-floorplan-and-placement)
  - [Step 4: Execute Floorplan and Placement](#step-4-execute-floorplan-and-placement)
  - [Step 5: Visualize Results in GUI](#step-5-visualize-results-in-gui)
- [Understanding Floorplanning](#understanding-floorplanning)
- [ORFS Directory Structure](#orfs-directory-structure)
- [Understanding the Visualizations](#understanding-the-visualizations)
- [Important Links](#important-links)
- [Reference](#reference)

---

## Overview

**OpenROAD** is an open-source, fully automated RTL-to-GDSII flow for digital integrated circuit (IC) design. It supports synthesis, floorplanning, placement, clock tree synthesis, routing, and final layout generation. OpenROAD enables rapid design iterations, making it ideal for academic research and industry prototyping.

---

## Why This Task is Important

After analyzing circuits at the transistor level, it's time to see how they are **physically realized on silicon**. This exercise introduces you to **OpenROAD**, a fully open-source RTL-to-GDSII flow used in both academic and industry research.

---

## Prerequisites

Before installing OpenROAD, ensure your system has the required dependencies:
```bash
sudo apt update
sudo apt install -y build-essential git cmake python3 python3-pip \
tcl-dev tk-dev libx11-dev libxrender-dev libxext-dev libboost-all-dev \
libeigen3-dev flex bison libreadline-dev
```

---

## Installation Steps

### Step 1: Clone OpenROAD Flow Scripts

Clone the main repository with all submodules:
```bash
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts
cd OpenROAD-flow-scripts
```

If you forgot `--recursive`, initialize submodules manually:
```bash
git submodule update --init --recursive
```

<img width="1177" height="617" alt="openroad_installation_1" src="https://github.com/user-attachments/assets/22680c9b-e900-4489-8382-40f67206f25f" />

<img width="1216" height="726" alt="openroad_installation_2" src="https://github.com/user-attachments/assets/33c56908-0560-459f-9adb-04bfb5121a96" />




Verify the repository cloned correctly:
```bash
ls
git submodule status
```

You should see directories like `flow/`, `tools/`, `docker/`, and a `README.md` file.



---

### Step 2: Build and Install OpenROAD

#### Option 1: Using Setup Script (Recommended)

Run the setup script:
```bash
sudo ./setup.sh
```

Build OpenROAD:
```bash
./build_openroad.sh --local
```



#### Option 2: Manual Build

Move into the flow directory:
```bash
cd flow
```

Set up the environment and install required dependencies:
```bash
sudo ./installers/BuildEnv.sh
make
```

---

### Step 3: Verify Installation

Source the environment:
```bash
source ./env.sh
```

Verify the tools are installed:
```bash
yosys -help
openroad -help
```

![Installation Step 3](<img width="1211" height="706" alt="yosys -help" src="https://github.com/user-attachments/assets/312e4997-6a2f-4b50-97dd-aca70c905f82" />)

![Installation Step 4](<img width="1207" height="410" alt="openroad -help" src="https://github.com/user-attachments/assets/4086aa76-2b96-4472-9dd9-8b3abc58e207" />)

Alternatively, launch OpenROAD directly:
```bash
./build/openroad
```

If the terminal shows the OpenROAD banner (OpenROAD vX.Y build...), your setup is complete ✅

---

## Running Floorplan and Placement

### Step 4: Execute Floorplan and Placement

Navigate to the flow directory:
```bash
cd flow
```

#### Option 1: Run Specific Stages

Execute only the Floorplan and Placement stages:
```bash
make DESIGN_CONFIG=./designs/asap7/gcd/config.mk floorplan
make DESIGN_CONFIG=./designs/asap7/gcd/config.mk place
```

#### Option 2: Run Complete Flow

Execute the full flow (stops before routing):
```bash
make
```

![Installation Step 5](<img width="1207" height="760" alt="flow_make" src="https://github.com/user-attachments/assets/af94743f-e843-4832-bc3a-8e4aed6e7de7" />)

**What Happens:**
- 🔹 **Floorplan Stage:** Defines core area, die dimensions, I/O placement
- 🔹 **Placement Stage:** Arranges standard cells to minimize delay & congestion
- ⛔ **STOPS Before Routing** (as required)

**Verification:**
- Core area and die dimensions are generated
- Standard cells are placed successfully

---

### Step 5: Visualize Results in GUI

Launch the graphical user interface (GUI) to visualize the final layout:
```bash
make gui_final
```

![GUI Final](<img width="1193" height="691" alt="make gui_final" src="https://github.com/user-attachments/assets/e9a406f2-6e0e-4542-848f-90ac7d1316dc" />)


## ⏱️ Timing Analysis in OpenROAD GUI

---

### 📘 Hold Slack Analysis
![Hold Slack](<img width="1217" height="777" alt="hold_slack" src="https://github.com/user-attachments/assets/538ad2b1-e388-4841-976f-519666306a36" />)

**Definition:**  
Hold slack = Data Arrival − (Clock Arrival + Hold Time)

**Meaning:**  
- ✅ Positive → Meets timing  
- ❌ Negative → Data changes too early  

**Observation:**  
- All paths show **positive slack (0.06–0.42 ns)**  
- No violations (no red bars)  
- **Design passes hold timing** — stable and safe for clock edges  

**Summary:**  
✔ Excellent hold margins  
✔ No timing fix needed  

---

### 🔴 Setup Slack Analysis
![Setup Slack](<img width="1047" height="636" alt="setup_slack" src="https://github.com/user-attachments/assets/e1d2b7a8-f055-419a-a58a-33322b70b515" />)

**Definition:**  
Setup slack = Required Time − (Data Arrival + Setup Time)

**Meaning:**  
- ✅ Positive → Meets timing  
- ❌ Negative → Data arrives late  

**Observation:**  
- ~10 paths **failing setup timing** (−0.06 ns → 0 ns)  
- Majority between 0.03–0.15 ns  
- Needs optimization (buffering, gate sizing, skew tuning)

**Summary:**  
⚠️ Setup violations present  
🔧 Requires timing closure before tape-out  

---

### 🧩 Comparison

| Metric | Hold | Setup |
|---------|------|-------|
| Status | ✅ Passing | ❌ Failing (~10 paths) |
| Slack Range | 0–0.42 ns | −0.06–0.15 ns |
| Design Health | Excellent | Needs optimization |

---

**Clock:** core_clock @ 0.46 ns (≈ 2.17 GHz)  
**Design:** GCD (Nangate45)  
**Next Step:** Fix setup slack → re-run STA until WNS ≥ 0 and TNS = 0




---

## Understanding Floorplanning

### What is Floorplanning?

Floorplanning is the **first major step** in the physical design flow, defining the **physical structure** of an integrated circuit (IC). It determines the **chip area, core boundary, placement of macros, I/O pins, and power grid**, providing a blueprint for placement and routing.

---

### Requirements Before Floorplanning

Before starting floorplanning, the following inputs and setup are essential:

#### 1. Technology Files
- **LEF (Library Exchange Format)** – Defines cell dimensions, metal layers, routing rules, sites, and obstructions
- **Tech LEF** – Describes process-level design rules and layer information
- **Liberty (.lib)** – Contains timing, power, and functional information for standard cells

#### 2. Logical Design Data
- **Synthesized Netlist (.v)** – Gate-level netlist output from synthesis (mapped to the standard cell library)
- **Design Constraints (.sdc)** – Includes clock definitions, input/output delays, and timing constraints

#### 3. Physical Design Inputs
- **I/O List or Pin Placement File (.io)** – Lists signal names and their preferred sides
- **Macro Placement Info (optional)** – For hard macros like SRAMs, PLLs, or IP blocks

#### 4. Environmental & Design Rules
- Core utilization target (e.g., 40–60%)
- Aspect ratio (height/width)
- Power planning strategy (ring, mesh, straps)
- Row height and site type defined in LEF

---

### Steps in Floorplanning

#### 1️⃣ Define Die and Core Area

Set total chip (die) and core dimensions using utilization and aspect ratio:
```tcl
initialize_floorplan -utilization 0.45 -aspect_ratio 1.0 -core_space {2 2 2 2} -site CORE
```

#### 2️⃣ Place Macros
- Fix positions of large blocks such as **memories, analog IPs, and PLLs** near the **core boundaries**
- Maintain adequate **routing channels** between macros and between macros and the core boundary
- Orient macros to align with power rails and minimize wirelength

#### 3️⃣ Place I/O Pins
- Define the positions of **input/output pads** around the **core perimeter**
- Follow signal grouping (e.g., data, control, power) for efficient routing
- Use **logical-to-physical pin mapping** if required by the design constraints

#### 4️⃣ Create Power Distribution Network (PDN)
- Generate **power (VDD)** and **ground (VSS)** rings around the core
- Add **metal straps** and **rails** to distribute power evenly

---

### Placement Overview

Placement determines exact cell positions within the floorplan.

**Steps:**
1. **Global placement** – minimize wirelength (HPWL)
2. **Detailed placement** – legalization, row alignment
3. **Filler insertion & optimization**

**Command Example:**
```tcl
place_design
```

---

## ORFS Directory Structure

### Main Directory Structure
```plaintext
OpenROAD-flow-scripts/
├── docker           -> Docker based installation, run scripts
├── docs             -> Documentation for OpenROAD or its flow scripts
├── flow             -> Files related to run RTL to GDS flow
├── jenkins          -> Regression tests for each build update
├── tools            -> All required tools to run RTL to GDS flow
├── etc              -> Dependency installer script and other files
└── setup_env.sh     -> Source file for OpenROAD environment
```

### Inside the `flow/` Directory
```plaintext
flow/
├── design           -> Built-in examples from RTL to GDS flow across different technology nodes
├── makefile         -> Automated flow runs through makefile setup
├── platform         -> Different technology node libraries, LEF files, GDS, etc.
├── tutorials        -> Tutorial files
├── util             -> Utility scripts
└── scripts          -> Flow scripts
```


---

## Understanding the Visualizations

### 🧱 Cell Placement Pattern

- **Rows:** Standard cells are aligned in neat horizontal rows to ensure power and ground rail continuity
- **Power Rails (VDD/VSS):** Located between alternate rows, supplying power and ground to the standard cells
- **White Spaces:** Reserved gaps between cells and rows to facilitate **routing channels** and reduce congestion
- **Color Coding:**
  - 🟦 **Buffers / Inverters** – small rectangular cells often repeated in timing paths
  - 🟩 **Logic Gates** – AND, OR, NAND, NOR cells in medium-sized blocks
  - 🟨 **Flip-Flops / Sequential Cells** – slightly larger cells, generally placed closer to the clock tree structure

This pattern ensures balanced power distribution, minimal delay, and optimal routing efficiency.

---

### 📊 Congestion Analysis

- 🟩 **Green:** Low congestion – easy routing with sufficient free tracks
- 🟨 **Yellow:** Medium congestion – routing density is moderate; tool may perform local optimization
- 🟥 **Red:** High congestion – routing is difficult; placement may need refinement or re-floorplanning
- 🟦 **Blue areas:** No congestion – typically reserved regions, blockages, or macro areas not used for routing

---

## Important Links

### 📘 Official Documentation
- [OpenROAD Main Documentation](https://openroad.readthedocs.io/en/latest/)
- [Floorplan Module – Initialize Floorplan](https://openroad.readthedocs.io/en/latest/main/src/ifp/README.html)
- [Placement Module – RePlAce / Detailed Placement](https://openroad.readthedocs.io/en/latest/main/src/gpl/README.html)

### 🧰 Tutorials & Guides
- [OpenROAD Flow Scripts Tutorial (RTL-to-GDS)](https://openroad-flow-scripts.readthedocs.io/en/latest/tutorials/FlowTutorial.html)
- [Getting Started with OpenROAD – Part 1 (Blog)](https://abdelrahmanhosny.me/tech/eda/2019-12-06-getting-started-with-openroad-1/)
- [OpenROAD Micro 2022 Tutorial (GitHub)](https://github.com/The-OpenROAD-Project/micro2022tutorial)
- [OpenROAD Flow Scripts Repository](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts)

### 💻 Source Code & Tools
- [OpenROAD GitHub Repository](https://github.com/The-OpenROAD-Project/OpenROAD)
- [OpenROAD Application Notes](https://openroad.readthedocs.io/en/latest/main/src/app/README.html)
- [TritonFP (Floorplan Generator)](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/TritonFP)

### 📚 Research & Overview
- [OpenROAD Project – Wikipedia](https://en.wikipedia.org/wiki/OpenROAD_Project)
- [OnDevTra Physical Design Overview](https://ondevtra.com/physicaldesign.html)
- [DAC 2022 OpenROAD Paper – Complete RTL-to-GDS Flow](https://doi.org/10.1145/3489517.3530632)

### 🧩 PDK & Example Designs
- [Sky130 Open PDK Support in OpenROAD Flow](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/master/flow/platforms/sky130hd)
- [Nangate45 Example Flow](https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts/tree/master/flow/platforms/nangate45)

---

## Reference

-(https://github.com/spatha0011/spatha_vsd-hdp/blob/main/Day14/README.md)

---

✅ **Installation Complete!** You can now explore the full RTL-to-GDSII flow using OpenROAD.
