# ESP32-C3 Migration Design Document
## Malenki-Nano Combat Robotics Receiver/ESC

### Executive Summary
This document tracks the technical decisions and implementation choices made during the ESP32-C3 migration project. It complements the PRD by documenting the "how" rather than the "what" of the project.

### Project Overview
- **Current System**: ATTiny16 + A7105 radio chip
- **Target System**: ESP32-C3 single chip (LCSC C2838500)
- **Primary Use**: Combat robotics (3-minute fight rounds)
- **Focus**: Technical decisions, trade-offs, and implementation choices

---

## 1. Architecture Decisions

### 1.1 Single-Chip Solution Decision
**Decision**: Use ESP32-C3 instead of separate radio + microcontroller
**Rationale**:
- **Cost**: Single extended inventory item vs two separate chips
- **Assembly**: Lower assembly fees (one placement vs two)
- **Complexity**: Reduced PCB complexity and routing
- **Performance**: 160MHz RISC-V vs 10MHz ATTiny16 (16x faster)

**Alternatives Considered**:
- **CC2500 + STM32F030**: Extended inventory for both parts
- **nRF24L01+ + STM32F030**: Extended inventory for radio
- **ESP32-C3**: Single extended inventory item

**Trade-offs**:
- **Pros**: Lower cost, simpler assembly, better performance
- **Cons**: Higher power consumption, larger footprint

### 1.2 Protocol Implementation Strategy
**Decision**: Software-defined radio approach
**Rationale**:
- **Flexibility**: Can implement multiple protocols on same hardware
- **Cost**: No additional radio chips needed
- **Maintainability**: Single codebase for all protocols

**Implementation Approach**:
- **AFHDS2A**: Port existing ATTiny16 code to ESP32-C3
- **ExpressLRS**: Use open source reference implementation
- **Betaflight**: Use open source flight controller code
- **FrSky R9**: Use open protocol specification

**Trade-offs**:
- **Pros**: Multi-protocol support, no additional hardware
- **Cons**: More complex software, potential timing issues

---

## 2. Hardware Design Decisions

### 2.1 Component Selection

#### 2.1.1 ESP32-C3 Variant
**Decision**: ESP32-C3-MINI-1 (LCSC C2838500)
**Rationale**:
- **Size**: Small footprint for combat robotics
- **Cost**: ~$2 USD, within budget
- **Features**: Built-in 2.4GHz radio, sufficient GPIO
- **Availability**: Standard inventory on LCSC

**Specifications**:
- **Clock**: 160MHz RISC-V processor
- **Memory**: 32KB RAM, 4MB flash
- **Radio**: Built-in 2.4GHz transceiver
- **GPIO**: 22 programmable pins
- **ADC**: 6 channels, 12-bit resolution

#### 2.1.2 Motor Driver Selection
**Decision**: Maintain existing motor driver architecture
**Rationale**:
- **Compatibility**: Existing designs proven in combat
- **Cost**: No additional development needed
- **Reliability**: Battle-tested components

**Current Drivers**:
- **Antweight**: DRV8837 (3x drivers)
- **Beetleweight**: DRV8243 (2x drivers)

##### DRV8231DSGR limits (quick reference)
- Operating supply (VM): 4.5 V to 33 V (recommended operating range)
- Peak output current: 3.7 A (peak; continuous current is thermally limited by PCB and airflow)
- Logic inputs: Compatible with 1.8 V, 3.3 V, and 5 V logic; input pins up to 5.5 V
- PWM: Supports high-frequency external PWM control (up to ~100 kHz typical usage)
- Protections: UVLO, OCP, and thermal shutdown
- Package: WSON-8, 2.0 × 2.0 mm (DSGR)
- Reference: see `ESP32-C3/Datasheets/drv8231 Datasheet.pdf` (TI DRV8231 product page)

#### 2.1.3 Power Management
**Decision**: Use existing power architecture with ESP32-C3 optimization
**Rationale**:
- **Compatibility**: Maintain existing battery configurations
- **Efficiency**: ESP32-C3 has better power management than ATTiny16
- **Reliability**: Proven power design

**Implementation**:
- **Voltage**: 1-4S LiPo (3.3V-16.8V)
- **Regulation**: 3.3V for ESP32-C3 and logic
- **Efficiency**: >90% at full load

### 2.2 PCB Design Decisions

#### 2.2.1 Combat Hardening
**Decision**: Enhanced shock and vibration resistance
**Rationale**:
- **Environment**: Combat robotics involves high impacts
- **Reliability**: 50G+ impact tolerance required
- **Performance**: Must survive 3-minute fight rounds

**Implementation**:
- **Component mounting**: Vibration-resistant mounting
- **PCB design**: Reinforced traces, thicker copper
- **Connector selection**: Locking connectors
- **Enclosure**: Impact-resistant housing

#### 2.2.2 Thermal Management
**Decision**: Active thermal monitoring and management
**Rationale**:
- **ESP32-C3**: Higher power consumption than ATTiny16
- **Combat**: High ambient temperatures in arenas
- **Reliability**: Thermal shutdown protection needed

**Implementation**:
- **Heat sinks**: Motor driver heat sinking
- **Thermal vias**: PCB thermal relief
- **Temperature monitoring**: ESP32-C3 internal sensor
- **Thermal shutdown**: Automatic protection

#### 2.2.3 EMI/RFI Considerations
**Decision**: Enhanced RF shielding and filtering
**Rationale**:
- **Radio performance**: Critical for combat robotics
- **Interference**: WiFi/Bluetooth in arenas
- **Reliability**: Must work in noisy environments

**Implementation**:
- **RF shielding**: Dedicated RF ground plane
- **Filtering**: Power and signal filtering
- **Grounding**: Proper ground plane design
- **ESD protection**: Input protection circuits

---

## 3. Software Architecture Decisions

### 3.1 Real-Time Requirements
**Decision**: Prioritize radio processing over other tasks
**Rationale**:
- **Latency**: <5ms end-to-end (radio to motor)
- **Reliability**: Packet loss <1% in normal conditions
- **Combat**: Fast response critical for survival

**Implementation**:
- **Radio processing**: <1ms latency
- **Motor control**: <100μs response
- **Failsafe**: <10ms activation
- **Telemetry**: Non-blocking operation

### 3.2 Protocol Implementation Strategy

#### 3.2.1 AFHDS2A Porting
**Decision**: Direct port from ATTiny16 code
**Rationale**:
- **Compatibility**: Maintain exact protocol behavior
- **Testing**: Existing transmitters work immediately
- **Risk**: Lower risk than rewriting from scratch

**Implementation**:
- **Packet timing**: 3.8ms intervals (maintained)
- **Frequency hopping**: 16 channels (ported)
- **Packet format**: 37-byte packets (exact)
- **FEC**: Forward Error Correction (ported)

#### 3.2.2 Modern Protocol Integration
**Decision**: Modular protocol architecture
**Rationale**:
- **Flexibility**: Easy to add new protocols
- **Maintainability**: Separate modules for each protocol
- **Testing**: Independent testing of each protocol

**Implementation**:
- **Protocol interface**: Common API for all protocols
- **Auto-detection**: Automatic protocol switching
- **Configuration**: User-selectable protocols
- **Fallback**: AFHDS2A as default

### 3.3 Memory Management
**Decision**: Static memory allocation with careful management
**Rationale**:
- **Reliability**: No dynamic allocation in combat
- **Performance**: Predictable memory usage
- **Safety**: No memory leaks or fragmentation

**Implementation**:
- **Protocol buffers**: Static allocation
- **Motor control**: Fixed-size buffers
- **Telemetry**: Pre-allocated structures
- **Configuration**: EEPROM emulation

### 3.4 Error Handling and Recovery
**Decision**: Comprehensive fault tolerance
**Rationale**:
- **Combat**: Must continue operating under partial failures
- **Reliability**: <1% failure rate in combat
- **Safety**: Automatic recovery from faults

**Implementation**:
- **Watchdog timer**: Hardware and software
- **Error recovery**: Automatic restart on faults
- **Memory protection**: Stack overflow protection
- **Fault logging**: Persistent error storage

---

## 4. Protocol-Specific Decisions

### 4.1 AFHDS2A Implementation
**Decision**: Software-defined radio emulation
**Rationale**:
- **Compatibility**: Exact protocol behavior
- **Hardware**: ESP32-C3 can handle timing requirements
- **Testing**: Use existing transmitters

**Technical Details**:
- **Frequency range**: 2400-2483MHz
- **Packet timing**: 3.8ms intervals
- **Channel hopping**: 16 channels
- **Packet format**: 37-byte packets
- **FEC**: Forward Error Correction

### 4.2 ExpressLRS Implementation
**Decision**: Use open source reference implementation
**Rationale**:
- **Performance**: <1ms latency
- **Range**: 10km+ capability
- **Community**: Active development and support

**Technical Details**:
- **Frequency hopping**: Advanced algorithms
- **Packet timing**: <1ms intervals
- **Telemetry**: Bidirectional communication
- **Configuration**: Over-the-air updates

### 4.3 Betaflight Implementation
**Decision**: Port flight controller protocol
**Rationale**:
- **Open source**: Well-documented protocol
- **Performance**: Optimized for real-time control
- **Community**: Large user base

**Technical Details**:
- **Packet format**: MSP protocol
- **Telemetry**: Comprehensive sensor data
- **Configuration**: Real-time parameter adjustment
- **Logging**: Flight data recording

### 4.4 FrSky R9 Implementation
**Decision**: Use open protocol specification
**Rationale**:
- **Range**: Long-range capability
- **Open protocol**: Well-documented
- **Compatibility**: Works with existing transmitters

**Technical Details**:
- **Frequency**: 900MHz operation
- **Range**: 10km+ capability
- **Telemetry**: Bidirectional communication
- **Configuration**: Real-time parameter adjustment

---

## 5. Performance Optimization Decisions

### 5.1 Timing Optimization
**Decision**: Hardware acceleration where possible
**Rationale**:
- **Latency**: Critical for combat robotics
- **Reliability**: Hardware is more predictable than software
- **Efficiency**: Lower power consumption

**Implementation**:
- **Hardware SPI**: Use ESP32-C3 hardware SPI
- **Hardware PWM**: Use hardware PWM for motor control
- **Hardware timers**: Use hardware timers for precise timing
- **DMA**: Use DMA for data transfers

### 5.2 Power Optimization
**Decision**: Dynamic frequency scaling and sleep modes
**Rationale**:
- **Battery life**: 3+ minutes under full load
- **Efficiency**: Optimize for combat duration
- **Thermal**: Reduce heat generation

**Implementation**:
- **Dynamic frequency**: Scale based on load
- **Sleep modes**: Deep sleep when possible
- **Peripheral shutdown**: Disable unused peripherals
- **Voltage scaling**: Adjust voltage based on load

### 5.3 Memory Optimization
**Decision**: Careful memory layout and usage
**Rationale**:
- **Performance**: Efficient memory access
- **Reliability**: No memory fragmentation
- **Size**: Minimize memory usage

**Implementation**:
- **Memory layout**: Optimized for access patterns
- **Buffer management**: Pre-allocated buffers
- **Cache usage**: Optimize cache hit rates
- **Stack management**: Careful stack usage

---

## 6. Testing and Validation Decisions

### 6.1 Combat Environment Testing
**Decision**: Real combat environment validation
**Rationale**:
- **Realistic**: Actual combat conditions
- **Reliability**: Validate under real stress
- **Performance**: Measure actual performance

**Implementation**:
- **Drop testing**: 50G+ impact testing
- **Vibration testing**: High-frequency vibration
- **Thermal testing**: High ambient temperature
- **EMI testing**: RF interference testing

### 6.2 Protocol Validation
**Decision**: Extensive testing with real transmitters
**Rationale**:
- **Compatibility**: Ensure exact protocol behavior
- **Reliability**: Validate with actual hardware
- **Performance**: Measure real-world performance

**Implementation**:
- **AFHDS2A**: Test with existing transmitters
- **ExpressLRS**: Test with ELRS transmitters
- **Betaflight**: Test with flight controllers
- **FrSky R9**: Test with R9 transmitters

### 6.3 Automated Testing
**Decision**: Comprehensive automated test suite
**Rationale**:
- **Reliability**: Catch issues early
- **Regression**: Prevent breaking changes
- **Performance**: Continuous performance monitoring

**Implementation**:
- **Unit tests**: Individual component testing
- **Integration tests**: Full system testing
- **Performance tests**: Latency and throughput
- **Stress tests**: Extended operation testing

---

## 7. Manufacturing and Assembly Decisions

### 7.1 Component Sourcing
**Decision**: LCSC for primary components
**Rationale**:
- **Cost**: Competitive pricing
- **Availability**: Good stock levels
- **Quality**: Reliable components

**Implementation**:
- **ESP32-C3**: LCSC C2838500
- **Passives**: Standard inventory components
- **Connectors**: Locking connectors for reliability
- **Enclosures**: Impact-resistant materials

### 7.2 PCB Manufacturing
**Decision**: 4-layer PCB with enhanced features
**Rationale**:
- **Signal integrity**: Better RF performance
- **Thermal management**: Dedicated thermal layers
- **Reliability**: More robust construction

**Implementation**:
- **Layer stack**: Signal, ground, power, signal
- **Copper thickness**: 2oz for current handling
- **Via types**: Through-hole and blind vias
- **Surface finish**: ENIG for reliability

### 7.3 Assembly Process
**Decision**: Automated assembly with manual inspection
**Rationale**:
- **Quality**: Consistent assembly
- **Cost**: Lower than manual assembly
- **Reliability**: Reduced human error

**Implementation**:
- **Pick and place**: Automated component placement
- **Reflow soldering**: Automated soldering
- **AOI**: Automated optical inspection
- **Manual inspection**: Critical component verification

---

## 8. Risk Mitigation Decisions

### 8.1 Technical Risks

#### 8.1.1 Software-Defined Radio Performance
**Risk**: May not meet timing requirements
**Mitigation**:
- **Early prototyping**: Test timing early
- **Hardware acceleration**: Use hardware where possible
- **Fallback modes**: Multiple implementation approaches
- **Performance monitoring**: Real-time performance tracking

#### 8.1.2 Thermal Management
**Risk**: ESP32-C3 may overheat under load
**Mitigation**:
- **Thermal simulation**: PCB thermal analysis
- **Heat sinking**: Adequate heat dissipation
- **Thermal monitoring**: Real-time temperature tracking
- **Thermal shutdown**: Automatic protection

#### 8.1.3 Shock Resistance
**Risk**: Components may fail under impact
**Mitigation**:
- **Component selection**: Shock-resistant components
- **Mounting**: Vibration-resistant mounting
- **Enclosure**: Impact-resistant housing
- **Testing**: Drop and vibration testing

### 8.2 Schedule Risks

#### 8.2.1 Protocol Porting Complexity
**Risk**: AFHDS2A porting may take longer than expected
**Mitigation**:
- **Incremental approach**: Port in phases
- **Parallel development**: Work on multiple protocols
- **Fallback plan**: Maintain existing system
- **Expert consultation**: Seek external expertise

#### 8.2.2 Hardware Design Complexity
**Risk**: PCB design may be more complex than expected
**Mitigation**:
- **Modular design**: Separate radio and control sections
- **Prototype testing**: Early hardware validation
- **Expert review**: PCB design review
- **Iterative approach**: Multiple design iterations

### 8.3 Resource Risks

#### 8.3.1 Component Availability
**Risk**: ESP32-C3 may become unavailable
**Mitigation**:
- **Alternative sources**: Multiple suppliers
- **Alternative parts**: Backup component options
- **Stock management**: Maintain adequate inventory
- **Design flexibility**: Easy component substitution

#### 8.3.2 Expertise Availability
**Risk**: Required expertise may not be available
**Mitigation**:
- **Knowledge transfer**: Document all decisions
- **Training**: Cross-train team members
- **External consultants**: Seek external expertise
- **Community resources**: Leverage open source community

---

## 9. Success Metrics and Validation

### 9.1 Performance Metrics
**Decision**: Comprehensive performance measurement
**Rationale**:
- **Validation**: Ensure requirements are met
- **Optimization**: Identify improvement opportunities
- **Comparison**: Compare with existing system

**Metrics**:
- **Latency**: <5ms end-to-end
- **Packet loss**: <1% in normal conditions
- **Range**: 50m+ in combat environment
- **Battery life**: 3+ minutes under full load

### 9.2 Reliability Metrics
**Decision**: Extensive reliability testing
**Rationale**:
- **Combat**: Must be reliable in combat
- **Safety**: Protect users and equipment
- **Reputation**: Build trust in the product

**Metrics**:
- **Shock resistance**: 50G+ impact tolerance
- **Temperature range**: -20°C to +85°C operation
- **Failure rate**: <1% failure rate in combat
- **Recovery time**: <10ms failsafe activation

### 9.3 Cost Metrics
**Decision**: Track cost throughout development
**Rationale**:
- **Budget**: Stay within $3-4 target
- **Competitiveness**: Maintain cost advantage
- **Profitability**: Ensure sustainable pricing

**Metrics**:
- **BOM cost**: <$3 total
- **Assembly cost**: Minimize assembly fees
- **Development cost**: Track development expenses
- **Total cost**: Complete system cost

---

## 10. Future Considerations

### 10.1 Scalability
**Decision**: Design for future expansion
**Rationale**:
- **Growth**: Support larger weight classes
- **Features**: Add new capabilities
- **Protocols**: Support additional protocols

**Implementation**:
- **Modular design**: Easy to add features
- **Protocol interface**: Standardized protocol API
- **Hardware expansion**: Additional GPIO and peripherals
- **Software architecture**: Extensible software design

### 10.2 Maintainability
**Decision**: Comprehensive documentation and testing
**Rationale**:
- **Longevity**: Support product for years
- **Updates**: Easy to add new features
- **Bug fixes**: Quick problem resolution

**Implementation**:
- **Code documentation**: Comprehensive code comments
- **Hardware documentation**: Complete schematics and layouts
- **Test procedures**: Automated and manual testing
- **Update procedures**: Over-the-air updates

### 10.3 Community Support
**Decision**: Open source approach where possible
**Rationale**:
- **Innovation**: Community contributions
- **Testing**: Broader testing base
- **Support**: Community support

**Implementation**:
- **Open protocols**: Use open source protocols
- **Documentation**: Public documentation
- **Community engagement**: Active community participation
- **Feedback integration**: Incorporate community feedback

---

## 11. Conclusion

This design document tracks the technical decisions made during the ESP32-C3 migration project. The decisions focus on:

1. **Reliability**: Combat robotics requires extreme reliability
2. **Performance**: Fast response times are critical for survival
3. **Cost**: Maintain competitive pricing
4. **Compatibility**: Support existing transmitters
5. **Future-proofing**: Design for future expansion

The single-chip ESP32-C3 solution provides the best balance of cost, performance, and reliability for combat robotics applications. The software-defined radio approach enables multi-protocol support while maintaining compatibility with existing AFHDS2A transmitters.

**Keywords for more details:** `technical decisions`, `implementation choices`, `trade-off analysis`
