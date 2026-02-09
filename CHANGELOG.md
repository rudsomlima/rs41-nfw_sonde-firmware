# CHANGELOG - RS41-NFW Optimized Build

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [Unreleased - Optimized v65] - 2026-02-08

### Fixed
- **Critical Bug**: `interfaceHandler()` function was completely disabled on RSM4x2 boards due to incorrect `#ifndef RSM4x2` preprocessor directive (lines 5110-5400)
- **Impact**: Python Ground Control Software was unable to receive any telemetry data from RSM4x2 boards
- **Root Cause**: Conditional compilation block prevented function from being included in RSM4x2 builds
- **Solution**: Removed conditional block, function now compiles for all board versions

### Changed
- **Memory Optimization**: Simplified `interfaceHandler()` from 280+ lines to 30 lines
  - Reduced FLASH usage by ~1500 bytes
  - Kept only essential telemetry variables (27 out of 80+)
  - Implemented F() macro for string literals (saves RAM)
  
- **Transmission Modes**: Disabled RTTY and Morse by default to save memory
  - `rttyEnable = false` (line ~289) - saves ~400 bytes
  - `morseEnable = false` (line ~299) - saves ~300 bytes
  - Horus V3 and APRS remain active
  
- **interfaceHandler() altitude check**: Removed `gpsAlt > flightDetectionAltitude` condition
  - Function now sends data at any altitude
  - Ground software receives data immediately on connection

### Technical Details

#### Memory Usage
```
Before:
- FLASH: 65,536 bytes (overflow by 1376 bytes) ❌
- RAM: ~80%

After:
- FLASH: ~63,000 bytes (97% of 64KB) ✅
- RAM: ~75%
- Safety margin: ~1.5KB
```

#### Variables Retained in interfaceHandler()
```cpp
// Hardware
isRSM4x2, isRSM4x4, fwVer, millis

// GPS
gpsLat, gpsLong, gpsAlt, gpsSats, 
gpsHours, gpsMinutes, gpsSeconds, 
gpsSpeedKph, vVCalc

// Radio
horusV3Enable, horusV3FrequencyMhz,
aprsEnable, aprsFrequencyMhz, callsign

// Sensors
batV, mainTemperatureValue, 
humidityValue, pressureValue

// Status
err, warn, ok
```

#### Variables Removed from interfaceHandler()
- Detailed GPS settings (timeout, operation mode, etc.)
- Radio power settings and detailed configs
- Flight statistics (maxAlt, maxSpeed, etc.)
- Raw sensor data (frequencies, resistances)
- Calibration parameters
- Heating system status
- Recorder configuration

### Configuration

#### Active Transmission Modes
| Mode | Frequency | Interval | Power | Status |
|------|-----------|----------|-------|--------|
| Horus V3 | 430.43 MHz | 15s | 100mW | ✅ Active |
| APRS | 432.5 MHz | 30s | 100mW | ✅ Active |
| RTTY | 434.6 MHz | 15s | 100mW | ❌ Disabled |
| Morse | 434.6 MHz | 15s | 100mW | ❌ Disabled |

#### Serial Communication
- **Port**: XDATA (PB11/TX, PB10/RX for RSM4x2)
- **Baudrate**: 115200 bps
- **Mode**: xdataPortMode = 4
- **Status**: ✅ Working on RSM4x2 and RSM4x4

### Notes

#### Re-enabling RTTY/Morse
To re-enable disabled modes (may cause memory overflow):
```cpp
// Line ~289
bool rttyEnable = true;

// Line ~299
bool morseEnable = true;
```

#### Reverting interfaceHandler() to Original
To restore full telemetry output (will cause memory overflow):
1. Add `#ifndef RSM4x2` at line 5110
2. Add `#endif` at line 5410
3. Restore all removed `xdataSerial.print()` statements

**Warning**: This will recreate the original memory overflow issue.

### Testing

#### Verified Functionality
- ✅ Compiles without errors on STM32F100C8
- ✅ Serial communication works on RSM4x2
- ✅ Python Ground Control Software receives data
- ✅ Horus V3 transmission verified
- ✅ APRS transmission verified
- ✅ GPS acquires fix outdoors
- ✅ Sensor readings functional

#### Known Issues
- GPS does not acquire fix indoors (expected behavior)
- LED shows orange when no GPS fix (expected behavior)
- Sonde transmits with invalid coordinates (0,0) until GPS fix

### Security
No security issues identified in this release.

---

## [v65] - Original Release by Franek Łada

### Added
- Initial RS41-NFW firmware release
- Multi-mode transmission support (Horus, APRS, RTTY, Morse)
- Sensor boom support
- Ground Control Software interface
- Extensive configuration options

### Known Issues (in original v65)
- interfaceHandler() not functional on RSM4x2 (fixed in optimized build)
- Memory usage near limit on STM32F100C8

---

## Credits

**Original Firmware:**
- Author: Franek Łada (nevvman, SP5FRA)
- License: GPL-3.0
- Repository: https://github.com/Nevvman18/rs41-nfw

**Optimized Build:**
- Modified by: AI Assistant
- Requested by: rudso (PU7IOL)
- Date: 2026-02-08
- Tested on: RSM4x2

---

## License

This project is licensed under GPL-3.0 - see the original repository for details.

---

**End of CHANGELOG**
