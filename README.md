## 3×3 Mean Filter IP
## Module Architecture
The **3×3 box blur IP** was designed around a pipelined streaming architecture with three main modules:

### **Input Line Buffers**:
Store rows of incoming pixel data. Three consecutive rows are buffered using circular line buffers to enable 3×3 window formation at each clock cycle.

**Key Features**:
- 512-entry memory (`line[511:0]`)
- Outputs 3 adjacent pixels (24-bit total)
- Controlled by write and read pointers
- **Inputs**: Clock, Reset, 8-bit Pixel Input, Valid, Read Enable  
- **Outputs**: 3 × 8-bit Pixels



### **Buffer Controller**: 
A finite state machine orchestrates read/write operations from the buffers, assembles 3×3 pixel windows, and ensures valid data is passed to the kernel in a streaming fashion.

**Functionality**:
- Uses 4 circular line buffers (lB0 to lB3)
- Tracks pixel and row indices
- Assembles 3×3 pixel windows (72-bit)
- Controls data flow via FSM
- Issues interrupt when full image is processed

**Key Signals**:
- `pixelCounter`, `currentWrLineBuffer`, `lineBuffDataValid`
- `currentRdLineBuffer`, `rd_line_buffer`
- `o_intr` interrupt output

### Kernel:
Performs the averaging filter operation over a 3×3 window by summing pixel values and dividing the result by 9. The kernel has a fixed latency of 2 cycles and operates in a fully pipelined manner.

**Operation**:
- Input: 72-bit window (9 pixels × 8 bits)
- Multiply each pixel by 1
- Sum and divide by 9 to compute average
- Output: 8-bit result
- Implemented with 2-cycle latency (multiply → sum/output)

Synthesized Schematic of the 3x3 IP block
![Synthesized Schematic](images/schematics_3x3.png)

The module was verified using a custom Verilog testbench, demonstrating efficient processing of a 512×512 grayscale image in approximately **2.65 milliseconds**. 

![Testbench Simulation](images/waveform_3x3.png)

The IP was packaged and integrated into a Vivado block design targeting the PYNQ-Z1 board.

![Block Design with implemented IP](images/block_design.png)

![Implementation Design](images/implemented_3x3.png)


## 7×7 Box Blur IP
To extend functionality, we developed a 7×7 box blur IP that supports **24-bit RGB images** and higher resolutions (up to **1024×1024**). Enhancements include:

- Support for larger kernel size and deeper window generation logic.
- Expanded buffer management to handle increased data bandwidth; 8 line buffers are instantiated and information from 7 buffers are fetched in a circular fashion.
- Color image compatibility with 24-bit pixel width.

Both **3×3** and **7×7** kernels are now implemented as separate IP blocks, packaged and bitstream-ready. However, more optimization is required for the design to improve synthesis performance and resource usage. Below are the LUT and Flip-flops utilization for both 3x3 and 7x7 designs.
