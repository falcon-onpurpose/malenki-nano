# ESP32-C3 Migration Product Requirements Document
## Malenki-Nano Combat Robotics Receiver/ESC

### Executive Summary
Migrate the existing ATTiny16 + A7105 radio design to ESP32-C3 single-chip solution while maintaining AFHDS2A compatibility and adding support for modern protocols. Focus on combat robotics reliability, shock resistance, and thermal management.

### Project Overview
- **Current System**: ATTiny16 + A7105 radio chip (~$2 total)
- **Target System**: ESP32-C3 single chip (~$2, LCSC C2838500)
- **Primary Use**: Combat robotics (3-minute fight rounds)
- **Key Requirements**: Reliability, shock resistance, thermal management

---

## 1. Functional Requirements

### 1.1 Protocol Compatibility
**Priority: Critical**
- **AFHDS2A Protocol**: Full compatibility with existing transmitters
  - 37-byte packets every 3.8ms
  - 16-channel frequency hopping
  - Forward Error Correction (FEC)
  - Telemetry support
  - Bind packet handling (0xbb, 0xbc)
  - Sticks packets (0x58)
  - Failsafe packets (0x56)

**Modern Protocol Support** (Phase 2):
- **ExpressLRS**: Open source, excellent performance, low latency (<1ms)
- **Betaflight**: Open source flight controller protocol
- **FrSky R9**: Good range, open protocol
- **Protocol auto-detection** and switching

### 1.2 Motor Control
**Priority: Critical**
- **3-channel ESC**: Left, Right, Weapon motors
- **PWM frequency**: 20kHz (reduced switching losses)
- **Current sensing**: Overcurrent protection
- **Failsafe behavior**: Motors off on signal loss
- **Smooth acceleration**: Prevent sudden movements

### 1.3 Combat Robotics Features
**Priority: High**
- **Shock resistance**: Component selection and mounting
- **Thermal management**: Heat dissipation design
- **Power efficiency**: Optimize for 3-minute fights
- **Fault tolerance**: Continue operation under partial failures
- **Quick recovery**: Restart after impacts

---

## 2. Non-Functional Requirements

### 2.1 Reliability & Robustness
**Priority: Critical**
- **Shock resistance**: 50G+ impact tolerance
- **Vibration resistance**: High-frequency vibration handling
- **Temperature range**: -20°C to +85°C operation
- **Humidity resistance**: Conformal coating recommended
- **Dust/particle resistance**: Sealed design where possible

### 2.2 Thermal Management
**Priority: High**
- **Power components**: Adequate heat sinking
- **ESP32-C3 thermal**: Monitor internal temperature
- **Thermal shutdown**: Automatic protection
- **Heat dissipation**: PCB design for thermal relief
- **Component spacing**: Prevent thermal coupling

### 2.3 Performance Requirements
**Priority: High**
- **Latency**: <5ms end-to-end (radio to motor)
- **Packet loss**: <1% under normal conditions
- **Range**: 50m+ in combat environment
- **Interference rejection**: Robust against WiFi/Bluetooth
- **Battery life**: 3+ minutes under full load

### 2.4 Size & Weight
**Priority: Medium**
- **PCB size**: Minimize footprint for small robots
- **Component height**: Low-profile components
- **Weight**: <50g total assembly
- **Mounting**: Multiple mounting options

---

## 3. Technical Specifications

### 3.1 ESP32-C3 Implementation
**Hardware:**
- **Chip**: ESP32-C3-MINI-1 (LCSC C2838500)
- **Clock**: 160MHz RISC-V processor
- **Memory**: 32KB RAM, 4MB flash
- **Radio**: Built-in 2.4GHz transceiver
- **GPIO**: 22 programmable pins
- **ADC**: 6 channels, 12-bit resolution

**Software-Defined Radio:**
- **AFHDS2A emulation**: Software implementation (ported from existing code)
- **ExpressLRS emulation**: Open source protocol implementation
- **Betaflight emulation**: Open source flight controller protocol
- **FrSky R9 emulation**: Open protocol implementation
- **Frequency hopping**: 16 channels, 2400-2483MHz
- **Packet timing**: 3.8ms intervals (AFHDS2A), <1ms (ExpressLRS)
- **FEC**: Forward Error Correction implementation

### 3.2 Power System
**Input:**
- **Voltage**: 1-4S LiPo (3.3V-16.8V)
- **Current**: 20A+ peak capability
- **Protection**: Reverse polarity, overvoltage, undervoltage

**Regulation:**
- **3.3V**: ESP32-C3 and logic
- **5V**: Optional servo power
- **Efficiency**: >90% at full load

### 3.3 Motor Drivers
**Specifications:**
- **Current**: 10A per channel continuous
- **Peak**: 20A per channel (30s)
- **Voltage**: Up to 16.8V
- **PWM**: 20kHz switching frequency
- **Protection**: Overcurrent, overtemperature, short circuit

---

## 4. Design Considerations

### 4.1 Combat Robotics Hardening
**Mechanical:**
- **Component mounting**: Vibration-resistant mounting
- **PCB design**: Reinforced traces, thicker copper
- **Connector selection**: Locking connectors
- **Enclosure**: Impact-resistant housing

**Electrical:**
- **EMI shielding**: RF shielding for radio
- **Filtering**: Power and signal filtering
- **Grounding**: Proper ground plane design
- **ESD protection**: Input protection circuits

### 4.2 Thermal Design
**Heat Sources:**
- **Motor drivers**: Primary heat source
- **ESP32-C3**: Moderate heat generation
- **Power regulation**: Secondary heat source

**Heat Management:**
- **Heat sinks**: Motor driver heat sinking
- **Thermal vias**: PCB thermal relief
- **Airflow**: Ventilation design
- **Temperature monitoring**: ESP32-C3 internal sensor

### 4.3 Software Architecture
**Real-time Requirements:**
- **Radio processing**: <1ms latency
- **Motor control**: <100μs response
- **Failsafe**: <10ms activation
- **Telemetry**: Non-blocking operation

**Reliability Features:**
- **Watchdog timer**: Hardware and software
- **Error recovery**: Automatic restart on faults
- **Memory protection**: Stack overflow protection
- **Fault logging**: Persistent error storage

---

## 5. Implementation Phases

### Phase 1: AFHDS2A Compatibility (Weeks 1-4)
**Goals:**
- Port existing ATTiny16 code to ESP32-C3
- Implement AFHDS2A protocol via software-defined radio
- Maintain exact packet timing and format
- Test with existing transmitters

**Deliverables:**
- Basic ESP32-C3 firmware
- AFHDS2A protocol implementation
- Motor control functionality
- Basic telemetry

### Phase 2: Combat Hardening (Weeks 5-8)
**Goals:**
- Implement shock/vibration resistance
- Add thermal management
- Optimize power efficiency
- Add fault tolerance features

**Deliverables:**
- Hardened PCB design
- Thermal management system
- Fault detection and recovery
- Power optimization

### Phase 3: Modern Protocols (Weeks 9-12)
**Goals:**
- Add ExpressLRS support (open source, excellent performance)
- Add Betaflight protocol support (open source)
- Add FrSky R9 support (open protocol)
- Implement protocol auto-detection
- Add advanced telemetry
- Optimize performance

**Deliverables:**
- Multi-protocol support (AFHDS2A, ExpressLRS, Betaflight, FrSky R9)
- Advanced telemetry
- Performance optimizations
- User configuration interface

### Phase 4: Testing & Validation (Weeks 13-16)
**Goals:**
- Combat environment testing
- Reliability validation
- Performance benchmarking
- User acceptance testing

**Deliverables:**
- Tested and validated system
- Documentation
- User manual
- Production-ready design

---

## 6. Risk Assessment

### 6.1 Technical Risks
**High Risk:**
- **Software-defined radio performance**: May not meet timing requirements
- **Thermal management**: ESP32-C3 may overheat under load
- **Shock resistance**: Components may fail under impact

**Medium Risk:**
- **Protocol compatibility**: AFHDS2A may have timing issues
- **Power efficiency**: May not meet battery life requirements
- **EMI interference**: Radio performance in noisy environment

**Low Risk:**
- **Component availability**: ESP32-C3 is widely available
- **Development complexity**: ESP32-C3 is well-documented

### 6.2 Mitigation Strategies
**Performance Risks:**
- **Prototype testing**: Early hardware validation
- **Thermal simulation**: PCB thermal analysis
- **Shock testing**: Drop and vibration testing

**Compatibility Risks:**
- **Protocol validation**: Extensive testing with real transmitters
- **AFHDS2A porting**: Ensure exact compatibility with existing code
- **Open source protocol integration**: ExpressLRS, Betaflight, FrSky R9
- **Timing optimization**: Hardware acceleration where possible
- **Fallback modes**: Multiple protocol implementations

---

## 7. Success Criteria

### 7.1 Functional Success
- [ ] AFHDS2A protocol 100% compatible (ported from existing code)
- [ ] ExpressLRS protocol support (open source)
- [ ] Betaflight protocol support (open source)
- [ ] FrSky R9 protocol support (open protocol)
- [ ] Motor control latency <5ms
- [ ] Packet loss <1% in normal conditions
- [ ] Range >50m in combat environment
- [ ] Battery life >3 minutes under full load

### 7.2 Reliability Success
- [ ] Survive 50G+ impacts
- [ ] Operate in -20°C to +85°C
- [ ] No thermal shutdown during 3-minute fights
- [ ] Automatic recovery from faults
- [ ] <1% failure rate in combat

### 7.3 Performance Success
- [ ] <$3 total BOM cost
- [ ] <50g total weight
- [ ] <100mm² PCB area
- [ ] >90% power efficiency
- [ ] <10ms failsafe activation

---

## 8. Resource Requirements

### 8.1 Hardware
- **ESP32-C3 development boards**: 5 units
- **Motor drivers**: 10 units for testing
- **Test equipment**: Oscilloscope, power analyzer
- **Combat robot test platform**: 1 unit

### 8.2 Software
- **ESP-IDF framework**: Development environment
- **Protocol analyzers**: AFHDS2A, ExpressLRS, Betaflight, FrSky R9 analysis tools
- **Open source protocol libraries**: ExpressLRS, Betaflight reference implementations
- **Simulation tools**: Thermal and EMI simulation
- **Testing framework**: Automated test suite

### 8.3 Expertise
- **ESP32-C3 development**: Embedded systems engineer
- **RF design**: RF engineer for antenna design
- **Thermal design**: Mechanical engineer
- **Combat robotics**: Domain expert

---

## 9. Timeline & Milestones

### Week 1-2: Foundation
- ESP32-C3 development environment setup
- Basic motor control implementation
- Initial AFHDS2A protocol analysis

### Week 3-4: Protocol Implementation
- AFHDS2A software-defined radio
- Packet timing and frequency hopping
- Basic telemetry implementation

### Week 5-6: Hardware Design
- PCB layout and thermal design
- Component selection and sourcing
- Prototype fabrication

### Week 7-8: Integration
- Hardware/software integration
- Basic functionality testing
- Performance optimization

### Week 9-10: Combat Hardening
- Shock and vibration testing
- Thermal management validation
- Fault tolerance implementation

### Week 11-12: Modern Protocols
- ExpressLRS implementation (open source)
- Betaflight protocol implementation (open source)
- FrSky R9 protocol implementation (open protocol)
- Protocol auto-detection
- Advanced features

### Week 13-14: Testing
- Combat environment testing
- Reliability validation
- Performance benchmarking

### Week 15-16: Finalization
- Documentation completion
- Production preparation
- User acceptance testing

---

## 10. Conclusion

The ESP32-C3 migration offers significant advantages for combat robotics applications while maintaining compatibility with existing AFHDS2A transmitters. The single-chip solution reduces assembly complexity and cost while providing the performance needed for modern protocols.

The focus on combat robotics reliability, thermal management, and shock resistance ensures the system will perform reliably in the demanding combat environment. The phased implementation approach allows for incremental validation and risk mitigation throughout the development process.

**Keywords for more details:** `thermal design`, `shock testing`, `protocol implementation`
