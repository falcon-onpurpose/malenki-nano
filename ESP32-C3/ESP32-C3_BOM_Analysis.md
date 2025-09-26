# ESP32-C3 BOM Analysis
## Malenki-Nano Combat Robotics Receiver/ESC Migration

### Executive Summary
Analysis of Bill of Materials (BOM) requirements for migrating from ATTiny16 + A7105 to ESP32-C3 single-chip solution. Examines ESP32-C3 datasheet requirements and compares with existing HV design BOM.

---

## 1. ESP32-C3 C2838500 Requirements

### 1.1 Core Specifications
- **Part Number**: ESP32-C3-MINI-1 (LCSC C2838500)
- **Package**: QFN-32 (5x5mm)
- **Power**: 3.3V operation
- **Clock**: 160MHz RISC-V processor
- **Memory**: 32KB RAM, 4MB flash
- **Radio**: Built-in 2.4GHz transceiver
- **GPIO**: 22 programmable pins
- **ADC**: 6 channels, 12-bit resolution

### 1.2 External Components Required
Based on ESP32-C3 datasheet analysis:

#### 1.2.1 Crystal Oscillator
- **Frequency**: 40MHz (required for ESP32-C3)
- **Package**: 2520-4Pin (2.5x2.0mm)
- **Current**: Seiko Epson Q24FA20H0051200 (C255947)
- **Alternative**: Any 40MHz crystal with 2520 footprint

#### 1.2.2 Power Supply Components
- **Voltage Regulator**: 3.3V LDO (replaces existing 3.3V reg)
- **Input Capacitors**: 10µF, 100nF for power filtering
- **Decoupling**: Multiple 100nF capacitors for noise suppression

#### 1.2.3 RF Components
- **Antenna**: Chip antenna (replaces A7105 antenna)
- **Matching Network**: LC components for antenna matching
- **RF Filtering**: Additional filtering for 2.4GHz operation

#### 1.2.4 Programming/Reset
- **Reset Circuit**: RC network for proper reset timing
- **Programming Interface**: UART0 for firmware updates
- **Boot Mode**: Pull-up/down resistors for boot configuration

---

## 2. Comparison with Existing HV Design

### 2.1 Current HV Design BOM (Malenki-HV)
```
Designator,Footprint,Quantity,Value,LCSC Part #
"C1, C2, C3, C5, C6",0603,5,10uF 25V,C96446
C10,0402,1,22pF,C1555
"C11, C14, C15",0402,3,10pF,C32949
"C12, C13, C16, C17, C22, C23, C4",0402,7,100nF,C307331
C18,0402,1,560pF,C1539
C19,0402,1,10nF,C15195
"C20, C21",0603,2,2.2uF,C23630
C9,0402,1,1pF,C1550
D1,0603,1,LED Blue,C72041
D2,0603,1,LED Red,C2286
L1,0402,1,2.7 nH,C76791
"L2, L3",0402,2,4.7nH,C341688
"R1, R5, R6",0402,3,470Ω,C25117
R2,0402,1,1kΩ,C11702
R25,0402,1,200Ω,C25087
R3,0402,1,100kΩ,C25741
"R4, R7, R8",0402,3,10kΩ,C25744
U1,a7105-QFN-20,1,A7105,C126376
U2,SOT-89-3,1,300mA Fixed 3.3V,C7434186
U3,VQFN-20-1EP_3x3mm_P0.4mm_EP1.7x1.7mm,1,ATTiny1616-MNR,C507118
"U4, U5, U6",WSON-8-1EP_2x2mm_P0.5mm_EP0.9x1.6mm,3,DRV8231DSGR,C3681197
Y1,Crystal_SMD_2520-4Pin_2.5x2.0mm,1,16mhz Seiko Epson Q24FA20H0051200,C255947
```

### 2.2 Key Changes Required

#### 2.2.1 Main IC Replacement
- **Remove**: A7105 (C126376) + ATTiny1616 (C507118)
- **Add**: ESP32-C3-MINI-1 (C2838500)
- **Net Change**: -2 ICs, +1 IC

#### 2.2.2 Crystal Oscillator
- **Current**: 16MHz crystal (C255947)
- **Required**: 40MHz crystal for ESP32-C3
- **Action**: Replace with C7301927 (confirmed)

#### 2.2.3 Power Supply
- **Current**: 3.3V LDO (C7434186)
- **Required**: Same 3.3V LDO (ESP32-C3 uses 3.3V)
- **Action**: Keep existing regulator

#### 2.2.4 RF Components
- **Remove**: A7105-specific RF components
- **Add**: ESP32-C3 RF components
- **Changes**:
  - Remove A7105 antenna matching
  - Add ESP32-C3 antenna matching
  - Update RF filtering

#### 2.2.5 Motor Drivers
- **Current**: DRV8231DSGR (C3681197) - 3x for HV
- **Required**: Same motor drivers
- **Action**: Keep existing motor drivers

---

## 3. Proposed ESP32-C3 BOM

### 3.1 Core Components

#### 3.1.1 Main IC
- **ESP32-C3-MINI-1**: LCSC C2838500
- **Package**: QFN-32 (5x5mm)
- **Cost**: ~$2 USD

#### 3.1.2 Crystal Oscillator
- **40MHz Crystal**: C7301927 (confirmed selection)
- **Package**: Updated footprint for ESP32-C3 compatibility
- **Frequency**: 40MHz (required for ESP32-C3)
- **Status**: Confirmed available on LCSC

#### 3.1.3 Power Supply
- **3.3V LDO**: Keep C7434186 (300mA Fixed 3.3V)
- **Input Capacitors**: Keep C96446 (10µF 25V)
- **Decoupling**: Keep C307331 (100nF) - may need more

### 3.2 RF Components

#### 3.2.1 Antenna
- **PCB Trace Antenna**: Reuse existing `malenki_antenna` design
- **Rationale**: 
  - Designed for 2.4GHz operation (perfect for ESP32-C3)
  - 50Ω impedance match
  - Zero cost (etched into PCB copper)
  - Proven in combat robotics
  - No additional assembly required
- **Footprint**: `malenki_antenna` (custom design)
- **Size**: ~15.5mm x 5.6mm (compact)
- **Reference**: TI Application Note SWRA117D

#### 3.2.2 Matching Network
- **Inductors**: Update L1, L2, L3 for ESP32-C3 RF output
- **Capacitors**: Update C10, C11, C14, C15 for 2.4GHz
- **Values**: Optimize for ESP32-C3 RF characteristics

**Current A7105 Values:**
- **L1**: 2.7nH (C76791 - TDK MLK1005S2N7ST000)
- **L2**: 4.7nH (C341688 - Murata LQW15AN4N7D10D)
- **L3**: 4.7nH (C341688 - Murata LQW15AN4N7D10D)
- **Footprint**: 0402 (1.0mm x 0.5mm)
- **Package**: SMD inductors

**ESP32-C3 Optimization Required:**
- **L1**: May need adjustment for ESP32-C3 RF output impedance
- **L2**: May need adjustment for antenna matching
- **L3**: May need adjustment for DC blocking/RF filtering
- **Testing**: Required to validate with ESP32-C3 RF characteristics

#### 3.2.3 RF Filtering
- **Add**: Additional filtering for 2.4GHz operation
- **Purpose**: Reduce interference and improve performance

### 3.3 Motor Control (Unchanged)

#### 3.3.1 Motor Drivers
- **DRV8231DSGR**: Keep C3681197 (3x for HV design)
- **Purpose**: H-Bridge drivers for motor control
- **Package**: WSON-8-1EP_2x2mm_P0.5mm_EP0.9x1.6mm

#### 3.3.2 Motor Control Components
- **Resistors**: Keep R1, R5, R6 (470Ω) for motor control
- **Capacitors**: Keep motor-related capacitors

### 3.4 Interface Components

#### 3.4.1 LEDs
- **Blue LED**: Keep C72041 (D1)
- **Red LED**: Keep C2286 (D2)
- **Purpose**: Status indication

#### 3.4.2 Programming Interface
- **Reset Circuit**: Add RC network for proper reset timing
- **Boot Mode**: Add pull-up/down resistors for boot configuration
- **UART**: Use UART0 for programming and debugging

### 3.5 Additional Components

#### 3.5.1 ESP32-C3 Specific
- **Reset Circuit**: RC network (R + C)
- **Boot Mode**: Pull-up/down resistors
- **Programming**: UART interface components

#### 3.5.2 Enhanced Features
- **Temperature Sensing**: ESP32-C3 internal sensor
- **WiFi**: Optional WiFi configuration
- **Bluetooth**: Optional Bluetooth for configuration

---

## 4. Cost Analysis

### 4.1 Component Cost Comparison

#### 4.1.1 Current HV Design
- **A7105**: ~$1.50 (C126376)
- **ATTiny1616**: ~$1.00 (C507118)
- **Total ICs**: ~$2.50

#### 4.1.2 ESP32-C3 Design
- **ESP32-C3**: ~$2.00 (C2838500)
- **Total ICs**: ~$2.00
- **Savings**: ~$0.50

#### 4.1.3 Assembly Cost
- **Current**: 2 IC placements
- **ESP32-C3**: 1 IC placement
- **Savings**: 1 placement fee

### 4.2 Total BOM Cost Estimate

#### 4.2.1 Core Components
- **ESP32-C3**: $2.00
- **Crystal C7301927**: $0.20
- **Power Supply**: $0.30
- **Passives**: $0.50
- **RF Components**: $0.30 (reuse antenna, optimize matching)
- **Motor Drivers**: $1.50 (3x DRV8231)
- **Total**: ~$4.80

#### 4.2.2 Cost Optimization
- **Target**: <$4.00 total
- **Strategies**:
  - Use standard inventory components
  - Optimize passive component selection
  - Consider alternative motor drivers

---

## 5. Component Selection Strategy

### 5.1 Priority 1: Core Functionality
- **ESP32-C3**: Must be C2838500 (confirmed available)
- **Crystal**: C7301927 (40MHz, confirmed available)
- **Power Supply**: Keep existing 3.3V LDO
- **Motor Drivers**: Keep existing DRV8231

### 5.2 Priority 2: RF Performance
- **Antenna**: 2.4GHz chip antenna
- **Matching Network**: Optimized for 2.4GHz
- **Filtering**: Adequate for combat environment

### 5.3 Priority 3: Cost Optimization
- **Passives**: Standard inventory components
- **Crystal**: Generic 40MHz if available
- **RF Components**: Standard 2.4GHz components

### 5.4 Priority 4: Enhanced Features
- **Temperature Monitoring**: ESP32-C3 internal sensor
- **Programming Interface**: UART-based
- **Debug Interface**: Optional WiFi/Bluetooth

---

## 6. Risk Assessment

### 6.1 Component Availability
- **ESP32-C3**: Extended inventory, but widely available
- **40MHz Crystal**: C7301927 confirmed available
- **RF Components**: Standard 2.4GHz components available

### 6.2 Performance Risks
- **RF Performance**: Software-defined radio vs dedicated RF IC
- **Timing**: ESP32-C3 timing vs ATTiny16 precision
- **Power Consumption**: ESP32-C3 higher power than ATTiny16

### 6.3 Cost Risks
- **ESP32-C3 Price**: May fluctuate
- **Assembly Cost**: Single IC reduces assembly fees
- **Total Cost**: Target <$4.00 achievable

---

## 7. Implementation Plan

### 7.1 Phase 1: Core BOM Validation
- **Verify ESP32-C3 availability**: C2838500
- **Test 40MHz crystal**: C7301927 operation
- **Validate power supply**: 3.3V operation
- **Test motor drivers**: DRV8231 compatibility

### 7.2 Phase 2: RF Component Selection
- **Validate antenna**: Test existing `malenki_antenna` with ESP32-C3
- **Optimize matching network**: Adjust L1, L2, L3 for ESP32-C3 RF output
- **Test RF performance**: Range and interference validation

### 7.3 Phase 3: Cost Optimization
- **Optimize passives**: Standard inventory components
- **Reduce assembly cost**: Single IC placement
- **Validate total cost**: <$4.00 target

### 7.4 Phase 4: Enhanced Features
- **Add temperature monitoring**: ESP32-C3 internal sensor
- **Implement programming interface**: UART-based
- **Add debug features**: Optional WiFi/Bluetooth

---

## 8. Conclusion

The ESP32-C3 migration offers significant advantages:

### 8.1 Cost Benefits
- **Single IC**: Reduces assembly complexity
- **Lower BOM**: ~$0.50 savings on ICs
- **Assembly Savings**: One placement vs two

### 8.2 Performance Benefits
- **16x Faster**: 160MHz vs 10MHz
- **More Memory**: 32KB RAM vs 1KB
- **Built-in Radio**: No external RF IC needed

### 8.3 Feature Benefits
- **Multi-protocol**: Software-defined radio
- **Enhanced Connectivity**: WiFi/Bluetooth options
- **Better Debugging**: More sophisticated tools

### 8.4 Risk Mitigation
- **Component Availability**: ESP32-C3 widely available
- **Performance Validation**: Extensive testing required
- **Cost Control**: Target <$4.00 achievable

The proposed BOM maintains compatibility with existing motor drivers while providing significant performance and feature improvements.

**Keywords for more details:** `component selection`, `cost analysis`, `RF design`
