# Changelog

All notable changes to the IBio project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- BRD.md: Business Requirements Document - comprehensive analysis of market, competition, cost targets, and feasibility for IBio FIDO2 security key project
- ADR-001: Architecture choice - ESP32-S3 + FreeRTOS + ESP-IDF
- ADR-002: MCU selection - ESP32-S3-WROOM-1-N8R8 as main controller
- ADR-003: Fingerprint sensor selection - FPM383C capacitive fingerprint module
- ADR-004: Security chip selection - ATECC608A-MAHDA-T for key storage
- ADR-005: Communication selection - USB + BLE dual-mode communication
- ADR-006: Power management selection - USB VBUS + AMS1117-3.3 LDO
- ADR-007: Storage selection - ESP32-S3 internal Flash + ATECC608A key storage
- HLD.md: High-level design document
- hardware-list.md: Hardware component list
- ESP32-S3-technical-specification.md: ESP32-S3 MCU technical specification (v1.0)
- FPM383C-technical-specification.md: FPM383C fingerprint sensor technical specification (v1.0)
- ATECC608A-technical-specification.md: ATECC608A security chip technical specification (v1.0)
- communication-technical-specification.md: USB + BLE communication module technical specification (v1.0)
- hardware-selection-report.md: Hardware selection comprehensive report integrating all ADR and technical specification documents

## [0.1.0] - 2026-04-06

### Added
- Project initialization
- Basic project structure
- README.md with project overview