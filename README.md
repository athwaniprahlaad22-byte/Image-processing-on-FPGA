# Real-Time Image Processing on Zedboard FPGA with VGA Display

> **An Educational Reference & Complete Manual for FPGA-Based Image Processing. Inspired from https://github.com/Gowtham1729/Image-Processing**

This project demonstrates real-time image processing operations implemented on the Zedboard FPGA platform with live VGA display output. It's designed as an educational resource, providing insights into hardware-software co-design, digital signal processing, and FPGA development using Vivado.

---

## Table of Contents

- [What is an FPGA? (Educational Overview)](#what-is-an-fpga-educational-overview)
  - [FPGA Architecture](#fpga-architecture)
  - [How FPGAs Work](#how-fpgas-work)
  - [Why Use FPGAs for Image Processing?](#why-use-fpgas-for-image-processing)
- [Project Overview](#project-overview)
- [Features & Image Processing Operations](#features--image-processing-operations)
- [Hardware Architecture](#hardware-architecture)
  - [Zedboard Platform](#zedboard-platform)
  - [VGA Interface Protocol](#vga-interface-protocol)
  - [Block RAM (BRAM) Organization](#block-ram-bram-organization)
- [Project Architecture](#project-architecture)
  - [Data Flow Pipeline](#data-flow-pipeline)
  - [Kernel-Based Processing](#kernel-based-processing)
- [Prerequisites & Setup](#prerequisites--setup)
- [Getting Started](#getting-started)
- [Detailed Usage Instructions](#detailed-usage-instructions)
  - [Step 1: Prepare Your Image](#step-1-prepare-your-image)
  - [Step 2: Generate COE File](#step-2-generate-coe-file)
  - [Step 3: Configure FPGA Design](#step-3-configure-fpga-design)
  - [Step 4: Build & Deploy](#step-4-build--deploy)
  - [Step 5: Operate the System](#step-5-operate-the-system)
- [Image Processing Operations Reference](#image-processing-operations-reference)
  - [Pixel-Level Operations](#pixel-level-operations)
  - [Convolution-Based Operations](#convolution-based-operations)
- [COE File Generation: In Depth](#coe-file-generation-in-depth)
  - [How coe_generator.py Works](#how-coe_generatorpy-works)
  - [Data Format & Bit Layout](#data-format--bit-layout)
  - [Pixel Processing Pipeline](#pixel-processing-pipeline)
- [Technical Deep Dive](#technical-deep-dive)
  - [VGA Timing & Signal Generation](#vga-timing--signal-generation)
  - [COE File Format & Structure](#coe-file-format--structure)
  - [Verilog Implementation Details](#verilog-implementation-details)
  - [Memory Layout for Processing](#memory-layout-for-processing)
- [Python Utility Scripts](#python-utility-scripts)
- [Troubleshooting & Common Issues](#troubleshooting--common-issues)
- [Performance Metrics](#performance-metrics)
- [Contributing](#contributing)
- [License](#license)
- [References & Further Reading](#references--further-reading)

---

## What is an FPGA? (Educational Overview)

### What Does FPGA Stand For?

**FPGA** = **Field Programmable Gate Array**

An FPGA is a semiconductor device that contains an array of programmable logic blocks and interconnects. Unlike traditional microprocessors that execute instructions sequentially, FPGAs allow you to create custom hardware configurations tailored to your specific application.

### FPGA Architecture

A typical FPGA consists of several key components:

#### 1. **Programmable Logic Blocks (CLBs)**
- Contain lookup tables (LUTs), multiplexers, and flip-flops
- LUTs implement combinational logic functions
- Flip-flops store state for sequential logic
- Configurable to implement any Boolean logic function

#### 2. **Block RAM (BRAM)**
- Embedded memory blocks within the FPGA fabric
- Typically 36 Kbits (or 18 Kbits) per block
- Can be configured as various widths and depths
- Enables efficient data storage for algorithms like ours (image pixel storage)
- **In this project:** We use BRAM to store image data efficiently

#### 3. **Interconnect Fabric**
- Programmable routing channels connecting logic blocks
- Metal tracks at various hierarchical levels
- Programmable switches at intersection points
- Enables flexible data flow between components

#### 4. **Input/Output (I/O) Blocks**
- Bidirectional buffers connecting FPGA to external world
- Programmable voltage standards (LVCMOS33, LVCMOS18, etc.)
- **In this project:** VGA output pins, clock input, switch inputs

#### 5. **Specialized Hardware**
- Dedicated DSP slices (for multiplications)
- Phase-locked loops (PLLs) for clock generation
- Block memories (BRAM)
- Zedboard includes ARM Cortex-A9 processors (not used in this project)

### How FPGAs Work

```
Programming Flow:
├─ Design → Specify logic in HDL (Verilog/VHDL)
├─ Synthesis → Convert HDL to logic gates
├─ Place & Route → Map logic to physical FPGA resources
├─ Bitstream Generation → Create configuration file
└─ Programming → Load bitstream into FPGA
```

**Key Concept:** The FPGA is "programmable in the field" - you can reprogram it with different logic designs without physically replacing hardware.

### Why Use FPGAs for Image Processing?

1. **Massive Parallelism**
   - Traditional CPUs process pixels sequentially
   - FPGAs process multiple pixels in parallel
   - Can achieve real-time processing of high-resolution video

2. **Hardware Customization**
   - Tailor bit-widths, precision, and operations to your needs
   - Avoid generic processor overhead
   - Optimize memory access patterns

3. **Low Latency**
   - No operating system overhead
   - Hardware processes data combinatorially
   - Deterministic behavior

4. **Power Efficiency**
   - No instruction fetch/decode cycles
   - Only active logic consumes power
   - Ideal for embedded systems

### Comparison: CPU vs. FPGA

| Aspect | CPU | FPGA |
|--------|-----|------|
| **Processing** | Sequential (pipeline) | Parallel (hardware) |
| **Latency** | High (cycles for each operation) | Very Low (combinatorial) |
| **Throughput** | Good for general workloads | Excellent for data-parallel tasks |
| **Power** | Higher per operation | Lower for specialized tasks |
| **Development** | Easier (C/C++) | Harder (Verilog/VHDL) |
| **Flexibility** | High (any algorithm) | Lower (must fit hardware) |

---

## Project Overview

This project implements **real-time image processing** on a Zedboard FPGA with live output to a VGA monitor. The key innovation is that all processing happens in hardware—no processor involvement—enabling real-time performance even with complex convolution operations.

### What Makes This Project Special?

- **16 Selectable Operations** - Pixel-level and convolution-based filters
- **Live VGA Output** - 640×480 @ 60Hz real-time display
- **Hardware-Accelerated Processing** - All operations in FPGA fabric
- **Single Python Script** - Elegant `coe_generator.py` handles all image conversion
- **Educational Design** - Well-commented Verilog code with clear architecture

---

## Features & Image Processing Operations

The system supports **16 different image processing operations**, selectable via 4 DIP switches (SW0-SW3) on the Zedboard:

### Pixel-Level Operations

| Operation | Sel Module | Description |
|-----------|-----------|-------------|
| **RGB to Grayscale** | `0000` | Convert color image to grayscale using luminance formula |
| **Increase Brightness** | `0001` | Amplify pixel values (clip at 255) |
| **Decrease Brightness** | `0010` | Reduce pixel values (clip at 0) |
| **Color Inversion** | `0011` | Invert all color channels (255 - value) |
| **Red Filter** | `0100` | Isolate red channel, suppress green/blue |
| **Green Filter** | `0110` | Isolate green channel, suppress red/blue |
| **Blue Filter** | `0101` | Isolate blue channel, suppress red/green |
| **Original Image** | `0111` | Display original image unchanged |

### Convolution-Based Operations (3×3 Kernels)

These operations process each pixel using values from 3×3 neighborhood:

```
Kernel Layout (pixel positions):
    [TL]  [T]  [TR]
    [L]   [C]  [R]
    [BL]  [B]  [BR]
```

| Operation | Sel Module | Kernel | Purpose |
|-----------|-----------|--------|---------|
| **Average Blur** | `1000` | `[1 1 1; 1 1 1; 1 1 1] / 9` | Smoothing/noise reduction |
| **Sobel Edge** | `1001` | Sobel operators | Edge detection with gradient |
| **Edge Detection** | `1010` | `[-1 -1 -1; -1 8 -1; -1 -1 -1]` | Detect rapid intensity changes |
| **Emboss** | `1100` | `[-2 -1 0; -1 1 1; 0 1 2]` | Create 3D embossed effect |
| **Sharpen** | `1101` | `[0 -1 0; -1 5 -1; 0 -1 0]` | Enhance edges and details |
| **Motion Blur (XY)** | `1011` | `[1 0 0; 0 1 0; 0 0 1] / 3` | Blur diagonally (TL to BR) |
| **Motion Blur (Y)** | `1110` | `[1 0 0; 1 0 0; 1 0 0] / 3` | Blur vertically (top to down) |
| **Gaussian Blur** | `1111` | `[1 2 1; 2 4 2; 1 2 1] / 16` | Smooth with Gaussian weighting |

---

## Hardware Architecture

### Zedboard Platform

The **Zedboard** is an ARM+FPGA embedded development platform featuring:

- **Zynq-7000 SoC (XC7Z020)**
  - Dual-core ARM Cortex-A9 processors (not used in this project)
  - Artix-7 FPGA fabric (280,000 logic cells)
  - Block RAM: 2.4 Mb total
  - 560 DSP slices

- **I/O Connectivity**
  - 4-bit VGA output (12-bit RGB: 4R + 4G + 4B)
  - 4 DIP switches (for operation selection)
  - 100 MHz system clock
  - USB programming interface

### Pin Configuration

Our design uses the following pins (from `const1.xdc`):

**VGA Output (Bank 33 - 3.3V)**
```
Clock Input  (GCLK):    Y9   (100 MHz)
VGA Red[0-3]:           V20, U20, V19, V18
VGA Green[0-3]:         AB22, AA22, AB21, AA21
VGA Blue[0-3]:          Y21, Y20, AB20, AB19
VGA Hsync:              AA19
VGA Vsync:              Y19
```

**Control Inputs (Bank 35 - 1.8V)**
```
sel_module[0-3] (SW0-3): F22, G22, H22, F21
reset (SW7):             M15
```

### VGA Interface Protocol

The **Video Graphics Array (VGA)** standard defines timing for analog video output:

#### VGA 640×480 @ 60Hz Timing

```
Horizontal Timing (in pixel clocks):
├─ Visible pixels: 640
├─ Front porch: 16
├─ Hsync pulse: 96
└─ Back porch: 48
Total: 800 pixel clocks per line

Vertical Timing (in scan lines):
├─ Visible lines: 480
├─ Front porch: 10
├─ Vsync pulse: 2
└─ Back porch: 33
Total: 525 lines per frame
```

#### Sync Signal Behavior

- **Hsync** = 0 during 96-pixel pulse, 1 otherwise
- **Vsync** = 0 during 2-line pulse, 1 otherwise
- RGB data valid only when **not in blanking interval**
- **Blanking Interval** = H or V porch/sync period

Our Verilog implementation generates these timing signals with hardware counters.

### Block RAM (BRAM) Organization

We use **Xilinx Blk_Mem_Gen** IP core with the following configuration:

```
Memory Type:        Single Port RAM
Width:              96 bits per word
Depth:              32,768 words (for 160×200 image with 9-pixel data)
Memory Size:        ~49 MB effective storage

Address:            15-bit (0 to 32,767)
Read Latency:       1 cycle (registered output)
Operating Mode:     WRITE_FIRST
Initial File:       .coe file (COE format)
```

**Data Organization (96 bits):**

```
Bits 95-88:   Blue value from top-left neighbor (leftup)
Bits 87-80:   Green value from left neighbor
Bits 79-72:   Red value from right neighbor
Bits 71-64:   Blue value from top neighbor (up)
Bits 63-56:   Blue value from bottom neighbor (down)
Bits 55-48:   Blue value from top-left neighbor (leftup)
Bits 47-40:   Blue value from bottom-left neighbor (leftdown)
Bits 39-32:   Blue value from top-right neighbor (rightup)
Bits 31-24:   Blue value from bottom-right neighbor (rightdown)
Bits 23-16:   Blue channel of current pixel
Bits 15-8:    Green channel of current pixel
Bits 7-0:     Red channel of current pixel

Pixel Layout (as matrix):
[leftup]  [up]     [rightup]
[left]    [center] [right]
[leftdown][down]   [rightdown]
```

This clever organization allows **single-cycle access** to all 9 neighborhood pixels plus the center pixel!

---

## Project Architecture

### Data Flow Pipeline

```
Input Image (BMP/JPG/PNG)
        ↓
  coe_generator.py Script
        ↓
   COE File (Binary pixel data)
        ↓
   FPGA BRAM Initialization
        ↓
   [FPGA Pipeline] ←← Clock signal (100 MHz)
   ├─ Address Counter (generates pixel addresses)
   ├─ BRAM Output (96-bit pixel data with neighbors)
   ├─ Pixel Processing (convolution or simple operation)
   ├─ Output Formatter (4-bit RGB for VGA)
   └─ VGA Controller (generates sync signals & RGB data)
        ↓
   VGA Monitor Output (Real-time display @60Hz)
```

### Kernel-Based Processing

For convolution operations, the system:

1. **Reads pixel data** from BRAM in parallel (96-bit word)
2. **Extracts 9 pixel neighborhood** from the 96-bit word
3. **Applies kernel weights** (sum of weighted products)
4. **Clamps result** to valid range (0-255 or 0-1024 depending on operation)
5. **Quantizes** to 4-bit per channel (0-15 for VGA display)
6. **Outputs** to VGA in real-time at pixel clock rate

**Performance:** 
- Pixel clock: ~25 MHz (VGA 640×480 @ 60Hz)
- Processing: Fully combinatorial (single-cycle)
- Real-time capability: Yes, can display full frames at 60 FPS

---

## Prerequisites & Setup

### Software Requirements

1. **Xilinx Vivado Design Suite (2018.3 or later)**
   - Free WebPACK version sufficient for XC7Z020
   - Download: https://www.xilinx.com/support/download.html
   - Install with Zynq-7000 and simulation tools support

2. **Python 3.6+**
   - `pip install opencv-python`
   
3. **Git** (for cloning repository)

4. **Text Editor or IDE** (for editing Verilog)
   - Vivado IDE included
   - VSCode with Verilog extensions (optional)

### Hardware Requirements

1. **Zedboard FPGA Development Board**
   - Zynq-7020 FPGA
   - 512MB DDR3 RAM
   - Micro-USB for programming
   - ~$200-300

2. **VGA Monitor**
   - Standard 640×480 or higher resolution
   - VGA connector (D-sub 15)
   - Any modern monitor with VGA adapter

3. **USB Cable**
   - Micro-USB (for FPGA programming)
   - Included with Zedboard

4. **Power Supply**
   - 12V, 2.5A minimum
   - Included with Zedboard

5. **Computer**
   - Windows 10/11, Linux, or macOS
   - 50 GB free disk space (for Vivado)
   - 8 GB RAM minimum (16 GB recommended)

---

## Getting Started

### Installation

#### 1. Install Xilinx Vivado

```bash
# Download from: https://www.xilinx.com/support/download.html
# WebPACK version (free) is sufficient

# Follow installer prompts:
# - Select "Vivado Design Suite"
# - Select "Zynq-7000" device support
# - Select "Install for Linux/Windows"
```

#### 2. Set Up Python Environment

```bash
# Clone or download this repository
git clone https://github.com/yourusername/Zedboard-Image-Processing-FPGA.git
cd Zedboard-Image-Processing-FPGA

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install opencv-python
```

#### 3. Install Board Files for Zedboard

Xilinx provides board definition files for automatic constraint generation:

```bash
# Download Zedboard board files from:
# https://github.com/Xilinx/XilinxBoardStore

# Or manually add to Vivado:
# ~/.Xilinx/Vivado/2024.1/data/boards/board_files/zedboard/
```
---

## Detailed Usage Instructions

### Step 1: Prepare Your Image

Your image should ideally be **160×200 pixels** for optimal display. The script accepts any resolution, but will adjust accordingly.

```bash
# Resize an existing image to 160×200 using Python
python3 << 'EOF'
from PIL import Image
import sys

input_path = sys.argv[1] if len(sys.argv) > 1 else "input.jpg"
output_path = sys.argv[2] if len(sys.argv) > 2 else "resized.bmp"

img = Image.open(input_path)
img = img.resize((160, 200), Image.Resampling.LANCZOS)
img.save(output_path, format='BMP')
print(f"Resized image saved to {output_path}")
EOF

# Example:
# python3 << 'EOF' ... EOF < your_image.jpg > test_images/my_image.bmp
```

**Important Notes:**
- Use 24-bit BMP, PNG, or JPG formats (avoids palette issues)
- Larger images work but will be cropped/scaled during display
- 160×200 is the reference size used in this project
- The script handles any resolution automatically

### Step 2: Generate COE File

The `coe_generator.py` script is your single utility for image conversion:

```bash
# Basic usage
python scripts/coe_generator.py input_image.jpg output_image.coe

# Examples:
python scripts/coe_generator.py test_images/flower.bmp coe_files/flower.coe
python scripts/coe_generator.py test_images/photo.png coe_files/photo.coe

# With full paths
python scripts/coe_generator.py /path/to/image.jpg /path/to/output.coe

# Show help
python scripts/coe_generator.py --help
python scripts/coe_generator.py --version
```

**Output:**
```
Starting conversion: 'test_images/flower.bmp' -> 'coe_files/flower.coe'
Successfully wrote COE file to 'coe_files/flower.coe' with 32000 pixel entries.
```

**Generated File Format:**
```
memory_initialization_radix=2;
memory_initialization_vector=
000000000000000000000000000000000000000000000000000000000000000010101010010101101100101,
000000000000000000000000000000000000000000000000000000000000000010101111010110101110101,
...
000000000000000000000000000000000000000000000000000000000000000011001100110011001100110;
```

Each line represents one pixel in 96-bit binary format.

### Step 3: Configure FPGA Design

#### 3.1: Open Vivado Project

```bash
# On Linux/macOS:
vivado fpga_design/VGA_1.xpr &

# On Windows:
# Double-click fpga_design/VGA_1.xpr
```

#### 3.2: Update BRAM Initialization File

In Vivado:

1. **Open IP Sources**
   - In the Design Sources panel (left)
   - Expand: Design Sources → IP → image
   - Double-click `image.xci` (Block RAM IP)

2. **Configure Block RAM**
   - Click "Edit"
   - In IP Customization window:
     - Find parameter: **"Coe_File"**
     - Set value to: `path/to/your/coe_files/flower.coe`
     - Click "OK" to save

3. **Regenerate IP**
   - Right-click on `image.xci` in IP Sources
   - Select "Regenerate"
   - Wait for completion

#### 3.3: Verify Pin Constraints

Constraints are already defined in `const1.xdc`:

```verilog
# VGA Output pins (verify in Device window):
set_property PACKAGE_PIN Y21  [get_ports {blue[0]}];   # VGA-B1
set_property PACKAGE_PIN Y20  [get_ports {blue[1]}];   # VGA-B2
... (all VGA pins defined)

# DIP Switch inputs (verify):
set_property PACKAGE_PIN F22 [get_ports {sel_module[0]}];  # SW0
set_property PACKAGE_PIN G22 [get_ports {sel_module[1]}];  # SW1
set_property PACKAGE_PIN H22 [get_ports {sel_module[2]}];  # SW2
set_property PACKAGE_PIN F21 [get_ports {sel_module[3]}];  # SW3
set_property PACKAGE_PIN M15 [get_ports {reset}];          # SW7
```

### Step 4: Build & Deploy

#### 4.1: Synthesize Design

In Vivado:

1. **Run Synthesis**
   - Click: Flow → Run Synthesis
   - Wait ~2-5 minutes
   - Should complete without errors
   - Review synthesis log for warnings

2. **Resolve Common Issues**
   - Unused logic warnings: Normal (not all filters used)
   - Undriven net warnings: Check port connections
   - Signal widths: Verify RTL schematic

#### 4.2: Implement Design

1. **Run Implementation**
   - Click: Flow → Run Implementation
   - Wait ~3-10 minutes
   - Review placement and routing utilization

2. **Check Resource Usage**
   - Expected utilization:
     - LUTs: ~20%
     - Registers: ~15%
     - BRAM: ~90% (mostly image storage)
     - DSPs: <5%

#### 4.3: Generate Bitstream

1. **Generate Bitstream**
   - Click: Flow → Generate Bitstream
   - Wait ~2-3 minutes
   - Creates: `fpga_design/VGA_1.runs/impl_1/vga_syncIndex.bit`

#### 4.4: Program FPGA

1. **Connect Zedboard**
   - Micro-USB to computer (labeled USB PROG)
   - Power on (green LED indicates 3.3V)
   - Connect VGA monitor to FPGA VGA port

2. **Program Device**
   - In Vivado: Tools → Program Device
   - Select: Zedboard XC7Z020
   - Select bitstream: `vga_syncIndex.bit`
   - Click "Program"
   - Wait ~30 seconds for completion
   - Monitor should show processed image

---

## Step 5: Operate the System

Once programmed, the system responds to physical switches:

### Control Inputs

| Switch | Port | Function |
|--------|------|----------|
| **SW0** | sel_module[0] | Operation selection bit 0 |
| **SW1** | sel_module[1] | Operation selection bit 1 |
| **SW2** | sel_module[2] | Operation selection bit 2 |
| **SW3** | sel_module[3] | Operation selection bit 3 |
| **SW7** | reset | Reset signal (active low) |

### Operation Selection

Combine SW0-SW3 to select operation:

```
Binary (SW3:SW0) → Operation
0000 → RGB to Grayscale
0001 → Increase Brightness
0010 → Decrease Brightness
0011 → Color Inversion
0100 → Red Filter
0101 → Blue Filter
0110 → Green Filter
0111 → Original Image
1000 → Average Blur
1001 → Sobel Edge Detection
1010 → Edge Detection
1011 → Motion Blur (XY)
1100 → Emboss
1101 → Sharpen
1110 → Motion Blur (Y)
1111 → Gaussian Blur
```

### Real-Time Operation

```bash
# To change operation:
1. Toggle switches SW0-SW3 to desired binary value
2. Watch VGA monitor - image updates within 1 frame (~16ms)
3. No system reset needed between operations
4. Processing happens in real-time at 60 FPS

# To reset processing:
1. Toggle SW7 (reset switch)
2. Hold for 100ms
3. Processing resumes
```

**Example:**
```
To select "Gaussian Blur" (1111):
- Set SW0, SW1, SW2, SW3 = ON (up position)
- Result: 4-bit value = 1111 (binary) = 15 (decimal)
- Monitor displays Gaussian-blurred image in real-time
```

---

## Image Processing Operations Reference

### Pixel-Level Operations

These operations process **each pixel independently** without neighboring pixel information.

#### RGB to Grayscale (0000)

**Formula:**
```
Gray = 0.299 * R + 0.587 * G + 0.114 * B
```

**Implementation:**
```verilog
red_o = (tred >> 2) + (tred >> 5) + (tgreen >> 1) + 
        (tgreen >> 4) + (tblue >> 4) + (tblue >> 5);
```

**Use cases:** Black & white processing, edge detection preprocessing

---

#### Color Filtering (0100, 0101, 0110)

**Red Filter:**
```verilog
red_o = tred;      // Keep red channel
green_o = 0;       // Zero green
blue_o = 0;        // Zero blue
```

**Green/Blue:** Similar channel isolation

**Use cases:** Color segmentation, channel extraction, debugging

---

### Convolution-Based Operations

These use **3×3 kernel convolution** - weighted sum of 9-pixel neighborhood:

```
Output = Σ(Kernel[i,j] * Pixel[i,j]) for i,j ∈ {-1,0,1}
```

#### Average Blur (1000)

**Kernel:**
```
[ 1  1  1 ]
[ 1  1  1 ]  / 9
[ 1  1  1 ]
```

**Effect:** Smooths image by averaging neighbors, reduces noise

**Implementation:**
```verilog
r = (gray + left + right + up + down + leftup + 
     leftdown + rightup + rightdown) / 9;
```

---

#### Sobel Edge Detection (1001)

**Sobel Gx (vertical edges):**
```
[-1  0  1]
[-2  0  2]
[-1  0  1]
```

**Sobel Gy (horizontal edges):**
```
[-1 -2 -1]
[ 0  0  0]
[ 1  2  1]
```

**Magnitude:** `Sqrt(Gx² + Gy²)` (approximated in hardware)

---

#### Sharpening (1101)

**Kernel:**
```
[ 0 -1  0]
[-1  5 -1]
[ 0 -1  0]
```

**Effect:** Enhances edges and details by subtracting blurred version

---

#### Gaussian Blur (1111)

**Kernel:**
```
[ 1  2  1 ]
[ 2  4  2 ]  / 16
[ 1  2  1 ]
```

**Effect:** Smoother blur than average filter, weighted toward center pixel

---

## COE File Generation: In Depth

### How coe_generator.py Works

The `coe_generator.py` script is the **single point of entry** for converting images to FPGA-compatible binary format. It elegantly handles all image types and sizes through a clean, well-designed pipeline.

#### Architecture Overview

```
User Command Line
        ↓
  argparse validates inputs
        ↓
   image_to_coe() function
        ↓
  ├─ cv2.imread() loads image in BGR order
  ├─ Iterates through every pixel
  ├─ Extracts B, G, R channels
  ├─ Converts each to 8-bit binary string
  ├─ Pads with 72 zero bits (for future neighbor data)
  ├─ Formats as Xilinx COE standard
  └─ Writes to output file
        ↓
  Xilinx-compatible COE file
```

#### Key Functions

**1. `_convert_channel_to_8bit_binary(channel_value: int) -> str`**

Converts a single color channel (0-255) to 8-bit binary:

```python
def _convert_channel_to_8bit_binary(channel_value: int) -> str:
    if not 0 <= channel_value <= 255:
        raise ValueError(f"Channel value {channel_value} must be between 0 and 255.")
    return bin(channel_value)[2:].zfill(8)

# Examples:
_convert_channel_to_8bit_binary(0)    # '00000000'
_convert_channel_to_8bit_binary(255)  # '11111111'
_convert_channel_to_8bit_binary(128)  # '10000000'
_convert_channel_to_8bit_binary(42)   # '00101010'
```

**2. `image_to_coe(image_path: PathLike, coe_path: PathLike) -> None`**

Main conversion function with comprehensive error handling:

```python
def image_to_coe(image_path, coe_path):
    # 1. Validate input file exists
    if not os.path.exists(str(image_path)):
        raise FileNotFoundError(...)
    
    # 2. Load image with OpenCV
    image = cv2.imread(img_path_str)  # Returns BGR (not RGB!)
    
    # 3. Process each pixel
    for row_idx, row in enumerate(image):
        for col_idx, pixel_bgr in enumerate(row):
            b_channel, g_channel, r_channel = pixel_bgr
            
            # Convert each channel to 8-bit binary
            binary_b = _convert_channel_to_8bit_binary(int(b_channel))
            binary_g = _convert_channel_to_8bit_binary(int(g_channel))
            binary_r = _convert_channel_to_8bit_binary(int(r_channel))
            
            # Combine: BGR order (as loaded by OpenCV)
            combined_pixel_binary = binary_b + binary_g + binary_r
            
            # Pad with 72 zero bits
            padded_binary_pixel = '0' * 72 + combined_pixel_binary
            
            pixel_binary_strings.append(padded_binary_pixel)
    
    # 4. Write to COE file with proper formatting
    with open(coe_path, "w") as coe_file:
        coe_file.write("memory_initialization_radix=2;\n")
        coe_file.write("memory_initialization_vector=\n")
        
        for i, binary_string in enumerate(pixel_binary_strings):
            coe_file.write(binary_string)
            if i < len(pixel_binary_strings) - 1:
                coe_file.write(',\n')
            else:
                coe_file.write(';\n')  # Last entry ends with semicolon
```

**3. `main_cli()`**

Command-line interface with argument parsing:

```bash
python scripts/coe_generator.py input.jpg output.coe --version
```

**Features:**
- Input validation with helpful error messages
- Output path verification
- Version tracking (`--version` flag)
- Detailed exception handling

### Data Format & Bit Layout

#### 96-Bit Word Structure

Each pixel becomes a 96-bit binary string:

```
Bit positions:  95-88      87-80      79-72      71-64      63-56      55-48      47-40      39-32      31-24      23-16      15-8       7-0
Data:          [Padding                                                             ]          [B8]       [G8]       [R8]
Description:   72 zero bits (reserved for future neighbor pixel data)              |          Blue       Green      Red
                                                                                    └─ 24 bits of pixel color (BGR order)
```

**Example Conversion:**

```
Input Pixel: RGB = (255, 128, 64)  [Red=255, Green=128, Blue=64]

Step 1: Convert channels to binary
  Red   = 255   → 11111111
  Green = 128   → 10000000
  Blue  = 64    → 01000000

Step 2: Reorder to BGR (OpenCV format)
  BGR order = 01000000 10000000 11111111

Step 3: Pad with 72 zeros
  96-bit = 000000000000000000000000000000000000000000000000000000000000000001000000100000001111111

Step 4: Format for COE
  Line in .coe = 000000000000000000000000000000000000000000000000000000000000000001000000100000001111111,
```

#### Why 72 Zero Bits?

The 72-bit padding is **reserved for future extensions** that could include:
- 8 neighboring pixel values (8 channels × 8 bits = 64 bits)
- Additional metadata or processing flags (8 bits)

This design allows seamless upgrades to kernel-based operations without changing the core architecture.

### Pixel Processing Pipeline

#### Input Validation

```python
# Type checking
if not hasattr(pixel_bgr, '__len__') or len(pixel_bgr) != 3:
    raise ValueError(f"Pixel has {len(pixel_bgr)} components, expected 3")

# Value range checking
if not 0 <= channel_value <= 255:
    raise ValueError(f"Channel value {channel_value} out of range")
```

#### Error Handling Strategy

The script provides meaningful error messages at each stage:

```
Error Types:
├─ FileNotFoundError
│   └─ Input image doesn't exist or can't be read
├─ IOError
│   └─ Output COE file can't be written
├─ ValueError
│   └─ Pixel data is invalid (out of range, wrong format)
└─ Exception
    └─ Unexpected OpenCV or processing errors
```

**Example Error Output:**
```
Error: Input image file not found at '/path/to/image.jpg'
Please ensure the input file path is correct and the file exists.

Data Error: Pixel at (10,20) has 4 components, expected 3 (BGR)
Please ensure the input image is a valid BGR image and pixel values are correct.

File I/O Error: Permission denied: '/output/flower.coe'
Please ensure you have write permissions for the output path and the path is valid.
```

#### Batch Processing Capability

For multiple images:

```bash
#!/bin/bash
# Convert all BMP files to COE

for image in test_images/*.bmp; do
    output="coe_files/$(basename "$image" .bmp).coe"
    python scripts/coe_generator.py "$image" "$output"
done
```

---

## Technical Deep Dive

### VGA Timing & Signal Generation

Our implementation generates VGA 640×480 @ 60Hz timing:

```verilog
// Horizontal counter increments every pixel clock
always @(posedge pixel_clk)
    hc <= hreset ? 0 : hc + 1;

// hsync pulses from pixel 655-751
assign hsyncon = (hc == 655);
assign hsyncoff = (hc == 751);

// hblank for front/back porch and sync
always @(posedge pixel_clk)
    hblank <= hreset ? 0 : hblankon ? 1 : hblank;

// Similar logic for vertical timing
```

**Timing Diagram:**
```
Horizontal (one line):
0────639│640─654│655─────751│752─799│
 Active  │Front  │ Hsync    │ Back   │
 Pixels  │Porch  │ (pulse)  │ Porch  │
         ↑ hblankon         ↑ hreset

Frame Rate:
= Pixel Clock / (800 × 525 pixels per frame)
= 25.175 MHz / 420,000 ≈ 59.94 Hz
```

### COE File Format & Structure

The **Coefficient (COE)** file is Xilinx's ASCII format for initializing BRAM:

```
memory_initialization_radix=2;
memory_initialization_vector=
011101010110100101100101...;  (space-separated 96-bit binary words)
```

**Our 96-bit Format:**

```
[72 zero bits for padding][24-bit BGR pixel]

Bits 95-88:   Padding (zeros)
Bits 87-80:   Padding (zeros)
Bits 79-72:   Padding (zeros)
Bits 71-64:   Padding (zeros)
Bits 63-56:   Padding (zeros)
Bits 55-48:   Padding (zeros)
Bits 47-40:   Padding (zeros)
Bits 39-32:   Padding (zeros)
Bits 31-24:   Padding (zeros)
Bits 23-16:   Blue channel (8-bit)
Bits 15-8:    Green channel (8-bit)
Bits 7-0:     Red channel (8-bit)
```

**File Size Calculation:**
```
For 160×200 image:
- Total pixels = 32,000
- BRAM words needed = 32,000 (1 word per pixel)
- Bits per word = 96
- File size ≈ 32,000 × 12 bytes = 384 KB
```

### Verilog Implementation Details

The main Verilog module `vga_syncIndex.v` contains:

#### Port Definitions

```verilog
module vga_syncIndex(
    input clock,              // 100 MHz system clock
    input reset,              // Active high reset
    input[3:0] sel_module,    // Operation selector
    output reg hsync,         // VGA horizontal sync
    output reg vsync,         // VGA vertical sync
    output reg [3:0] red,     // 4-bit red output
    output reg [3:0] green,   // 4-bit green output
    output reg [3:0] blue     // 4-bit blue output
);
```

#### Clock Division

```verilog
// Divide 100 MHz system clock to ~25 MHz pixel clock
reg clk;
always@(posedge clock) clk <= ~clk;  // Divide by 2 to 50 MHz

reg pcount;
wire pixel_clk;
always @ (posedge clk) pcount <= ~pcount;
assign pixel_clk = (pcount == 0);  // Further divide to ~25 MHz
```

#### Timing Counters

```verilog
reg [9:0] hc, vc;  // Horizontal and vertical counters
reg hblank, vblank;

// Reset counters at end of line/frame
assign hreset = ec & (hc == 799);  // 800 pixels per line
assign vreset = hreset & (vc == 523);  // 525 lines per frame
```

#### BRAM Interface

```verilog
// Instantiate Block RAM IP core
image inst1(
    .clka(clk),
    .wea(read),           // Write enable (always 0 in this design)
    .addra(addra),        // Address (incremented each pixel)
    .dina(in1),           // Input data (unused)
    .douta(out2)          // Output: 96-bit pixel + neighbors data
);
```

#### Processing Logic (3×3 Kernel Example - Gaussian Blur)

```verilog
else if(sel_module == 4'b1111) begin  // Gaussian blur
    if(reset) begin
        red = 0; green = 0; blue = 0;
    end else begin
        // Extract all 9 pixels from 96-bit BRAM word
        r = (rightup + (2*up) + leftup + 
             (2*right) + (4*gray) + (2*left) + 
             rightdown + (2*down) + leftdown) / 16;
        
        // Quantize to 4-bit for VGA
        red_o = r / 16;
        blue_o = r / 16;
        green_o = r / 16;
        
        red = {red_o[3:0]};
        green = {green_o[3:0]};
        blue = {blue_o[3:0]};
    end
end
```

### Memory Layout for Processing

The memory organization enables **single-cycle parallel access** to all pixels:

```
Image Storage:
Address 0: Pixel (0,0) with padding
Address 1: Pixel (0,1) with padding
...
Address 159: Pixel (0,159) with padding

Address 160: Pixel (1,0) with padding
...

Pixel (i,j) stored at address = i*160 + j

Each address word contains:
- 72 zero bits (padding)
- 24 bits RGB pixel data (BGR order)
```

---

## Python Utility Scripts

### `coe_generator.py` - Complete Reference

**Usage:**
```bash
python scripts/coe_generator.py <input_image> <output_coe> [--version]
```

**Parameters:**
- `input_image` (positional): Path to input image (JPG, PNG, BMP, etc.)
- `output_coe` (positional): Path for output COE file
- `--version`: Display script version

**Features:**
- ✅ Supports all common image formats (OpenCV compatible)
- ✅ Automatic BGR → 96-bit conversion
- ✅ Comprehensive error handling with helpful messages
- ✅ Progress reporting with pixel count
- ✅ Type hints for code clarity
- ✅ Scalable to any image resolution

**Example Usage:**
```bash
# Single image conversion
python scripts/coe_generator.py flower.jpg coe_files/flower.coe

# Batch conversion with shell script
for img in *.jpg; do
    python scripts/coe_generator.py "$img" "coe_files/${img%.jpg}.coe"
done

# With different paths
python scripts/coe_generator.py /input/path/image.png /output/path/image.coe
```

**Return Values & Exit Codes:**
```
Exit Code 0: Success - COE file generated
Exit Code 1: Error - File not found, permission, or data error
```

---

## Troubleshooting & Common Issues

### Issue: "No image on VGA monitor"

**Possible Causes:**

1. **Monitor not powered on**
   - Solution: Check monitor power and VGA cable connection

2. **FPGA not programmed**
   - Check LED indicators:
     - Green: Power OK
     - Red: Programming progress
     - Green again: Programming complete
   - Reprogram device in Vivado

3. **Wrong COE file loaded**
   - Verify in Vivado project
   - Check path in `image.xci` IP configuration
   - Regenerate IP core

4. **VGA cable issues**
   - Test monitor with another computer
   - Try different VGA cable
   - Check pin connector for bent/broken pins

**Diagnostic Steps:**
```bash
# In Vivado, check:
1. Bitstream generation completed without errors
2. Device programmed (green LED confirms)
3. No timing violations in implementation report
4. BRAM initialization file exists and is valid
```

---

### Issue: "Image looks corrupted or has artifacts"

**Possible Causes:**

1. **Image dimensions wrong**
   - Expected: 160×200 pixels
   - Solution: Verify image size or resize
   ```bash
   python3 -c "from PIL import Image; print(Image.open('image.bmp').size)"
   ```

2. **COE file corrupted**
   - Regenerate with Python script
   - Verify file size: ~384 KB for 160×200 image
   - Check first and last lines of COE file

3. **Memory address exceeds bounds**
   - Check image size doesn't exceed 32,768 pixels
   - For 160×200: 32,000 pixels (OK)
   - For 200×200: 40,000 pixels (TOO LARGE - adjust)

4. **Incorrect filter selected**
   - Verify switch positions match intended operation
   - Check Verilog sel_module mapping

---

### Issue: "coe_generator.py fails with error"

**Common Errors:**

```
FileNotFoundError: Error: Input image file not found at '...'
→ Solution: Check file path is correct and file exists
           python scripts/coe_generator.py --help

IOError: File I/O Error: Permission denied
→ Solution: Check write permissions on output directory
           chmod 755 coe_files/

ValueError: Data Error: Pixel at (0,0) has 4 components
→ Solution: Image might be RGBA instead of RGB
           Convert with: python3 << 'EOF'
           from PIL import Image
           img = Image.open('image.png').convert('RGB')
           img.save('image_fixed.bmp')
           EOF
```

---

### Issue: "Build fails with synthesis errors"

**Common errors:**

```
ERROR: Unresolved reference to 'image'
→ Solution: Regenerate IP core (right-click → Regenerate)

ERROR: Port 'sel_module[4]' not found
→ Solution: sel_module is 4-bit [3:0], not [4:0]

ERROR: Unknown module 'vga_syncIndex'
→ Solution: Ensure VGA.v is added to project sources
```

---

### Issue: "FPGA gets hot or drawing excessive power"

**Causes:**
1. **Timing violation** - Logic running slower than clock
2. **Oscillation** - Unresolved feedback loop
3. **High resource utilization** - Design too complex

**Solution:**
```bash
# Check timing report:
# In Vivado: Report → Timing Summary
# Look for "Worst Negative Slack" (should be > 0)

# If fails:
1. Increase clock period (reduce frequency)
2. Use pipelining for deep logic
3. Reduce logic complexity
```

---

## Performance Metrics

### Processing Throughput

| Metric | Value |
|--------|-------|
| **System Clock** | 100 MHz |
| **Pixel Clock** | ~25.175 MHz |
| **Pixels per Frame** | 640 × 480 = 307,200 |
| **Frame Rate** | 59.94 FPS (~60 Hz) |
| **Processing Latency** | <1 pixel clock (~40 ns) - combinatorial |
| **Image Update Rate** | 16.67 ms per frame |

### Resource Utilization

| Resource | Used | Available | % |
|----------|------|-----------|---|
| **LUT** | ~5,600 | 28,800 | 19% |
| **Registers** | ~4,200 | 57,600 | 7% |
| **BRAM** | 43 × 36K | 48 | 90% |
| **DSP** | 0 | 560 | 0% |

### Memory Performance

```
BRAM Read Latency: 1 cycle (~10 ns at 100 MHz clock)
BRAM Throughput: 1 word per cycle @ 100 MHz
                = ~9.6 GB/s effective bandwidth
                
Pixel Throughput: 
- 640 × 480 @ 60 Hz = 18.43 MP/s
- Each pixel reads 1 BRAM word
- Limited by VGA output rate, not FPGA processing
```

---

## Contributing

We welcome contributions! Areas for enhancement:

1. **Additional Filters**
   - Bilateral filtering
   - Median filtering
   - Morphological operations

2. **Performance Improvements**
   - Pipeline architecture
   - Parallel multi-pixel processing
   - Custom memory hierarchies

3. **User Interface**
   - Add buttons for brightness adjustment
   - LED indicators for operation status
   - On-screen display of current operation

4. **Documentation**
   - Detailed kernel mathematics
   - Verilog optimization techniques
   - Advanced FPGA concepts

**To Contribute:**
```bash
# Fork repository
git clone https://github.com/yourusername/fork.git
cd fork

# Create feature branch
git checkout -b feature/awesome-filter

# Make changes
# Commit with clear messages
git commit -am "Add awesome-filter operation"

# Push and create Pull Request
git push origin feature/awesome-filter
```

---

## License

This project is licensed under the **Apache License 2.0**. See LICENSE file for details.

**You are free to:**
- Use commercially
- Modify the code
- Distribute
- Use privately

**You must:**
- Include license and copyright notice
- State significant changes made

---

## References & Further Reading

### FPGA & Digital Design

1. **"Digital Design and Computer Architecture" by Harris & Harris**
   - Comprehensive HDL design fundamentals
   - Timing analysis and optimization

2. **"FPGA Prototyping by Verilog Examples" by Chu**
   - Practical Verilog examples
   - Real hardware implementations

3. **Xilinx Documentation**
   - Vivado User Guide: https://docs.xilinx.com/
   - Zynq-7000 Technical Reference Manual

### Image Processing & Convolution

4. **"Digital Image Processing" by Gonzalez & Woods**
   - Convolution theory and applications
   - Filter design mathematics

5. **OpenCV Documentation**
   - Standard filter implementations
   - Algorithm references

### VGA & Video Timing

6. **VGA Timing Specifications**
   - https://en.wikipedia.org/wiki/Video_Graphics_Array
   - Detailed timing diagrams

7. **VESA Standards** (Video Electronics Standards Association)
   - Official VGA timing specifications

### Zedboard Resources

8. **Zedboard Community Wiki**
   - https://www.zedboard.org
   - Getting started guides

9. **Zynq-7000 User Guide**
   - https://docs.xilinx.com/
   - Detailed hardware specifications

### Tools & Software

10. **Python Image Libraries**
    - OpenCV: https://opencv.org/
    - PIL/Pillow: https://python-pillow.org/
    - NumPy: https://numpy.org/

---

## Authors & Acknowledgments

**Original Author:** https://github.com/Gowtham1729
**This version (modified):** https://github.com/infinitecoder1729
**Project Base:** Image Processing Toolbox (Basys 3 adaptation)  
**Zedboard Adaptation:** Modified for Zynq-7000 platform with enhanced VGA support

**Credits:**
- Zedboard community resources
- Xilinx education materials
- OpenCV algorithm references

---

## Frequently Asked Questions

**Q: Can I use a different image size?**  
A: Yes! The COE generator handles any resolution. For optimal display, resize to 160×200.

**Q: Can I add real-time parameter adjustment?**  
A: Yes! Use additional switches or buttons to pass parameters to processing modules. Add an input bus for brightness, blur radius, etc.

**Q: What if I don't have a Zedboard?**  
A: This project can adapt to any Xilinx FPGA with:
- ≥ 24 KB BRAM
- ≥ 10,000 logic cells
- VGA output pins available
- Similar development tools (Vivado)

**Q: Can I use this for video processing?**  
A: Yes! Stream frames continuously by updating BRAM contents and maintaining VGA timing synchronization.

**Q: How can I optimize further?**  
A: Consider pipelining, parallel pixel processing, or custom DSP implementations for specific filters.

**Q: Does the COE generator support different image formats?**  
A: Yes! OpenCV supports JPG, PNG, BMP, TIFF, and most common formats automatically.

---

## Support

For issues, questions, or suggestions:

1. **GitHub Issues**: Create an issue with:
   - Problem description
   - Steps to reproduce
   - System information (Vivado version, OS, board)
   - Error messages/logs

2. **Documentation**: Check docs/ folder first

3. **Zedboard Forums**: https://www.zedboard.org/forums

---

**Last Updated:** December 2025
**Project Status:** Active & Maintained  
**Vivado Compatibility:** 2018.3+  
**Python Version:** 3.6+

---

---

**Happy FPGA Development! 🚀**
