# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Global Coding Practices

## General Best Practices
- Use established conventions and follow the existing codebase patterns
- Write clean, maintainable code with meaningful variable and function names
- Prefer composition over inheritance where appropriate
- Handle errors gracefully and provide meaningful error messages
- Write tests for new functionality and bug fixes

## C++ Specific
- Use references instead of passing by value when it makes sense (for non-primitive types and when the parameter won't be null)
- Prefer `const` references for read-only parameters
- Use smart pointers (`std::unique_ptr`, `std::shared_ptr`) for memory management
- Follow RAII principles for resource management
- Use `auto` for type deduction when it improves readability

## Comments
- Limit comments to only what is needed to understand the code
- Focus on explaining *why* something is done, not *what* is being done
- Document complex algorithms and business logic
- Avoid obvious comments that just restate the code
- Keep comments up-to-date with code changes

# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.

## Python Protocol Gateway

This is a Python-based service that reads data via Modbus RTU or other protocols and translates the data for MQTT. It serves as a general-purpose protocol gateway for industrial automation and solar monitoring systems.

### Key Commands

**Development and Testing:**
```bash
# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt

# Run linting (uses flake8)
flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics

# Run code formatting and additional linting (uses ruff)
ruff check .
ruff format .

# Run tests
pytest
```

**Running the Application:**
```bash
# Run with default config (config.cfg)
python3 -u protocol_gateway.py

# Run with specific config file
python3 -u protocol_gateway.py config.cfg

# Run via entry point (after pip install)
protocol-gateway
# or
ppg
```

**Docker:**
```bash
# Build image
docker build . -t protocol_gateway

# Run with device access
docker run --device=/dev/ttyUSB0 protocol_gateway

# Run with config volume
docker run -v $(pwd)/config.cfg:/app/config.cfg --device=/dev/ttyUSB0 hotn00b/pythonprotocolgateway
```

### Architecture Overview

**Core Components:**
- `protocol_gateway.py` - Main entry point and configuration parser
- `classes/protocol_settings.py` - Protocol configuration and registry mapping
- `classes/transports/` - Transport layer implementations (MQTT, Modbus RTU/TCP, InfluxDB, etc.)

**Protocol System:**
- Protocols are defined in `protocols/` directory with JSON config files and CSV registry maps
- Each protocol has input/holding registry maps defining data structure and variable names
- Support for multiple device types: Growatt, EG4, Sigineer, SOK, PACE-BMS, SMA, etc.

**Transport Architecture:**
- Base transport class in `classes/transports/transport_base.py`
- Input transports: Modbus RTU/TCP/UDP, CAN Bus, Serial protocols
- Output transports: MQTT, InfluxDB, JSON file output
- Gateway mode: Read from one transport, output to another

**Configuration:**
- Main config in `config.cfg` (copy from `config.cfg.example`)
- Protocol version specified as `protocol_version` in config
- Variable filtering via `variable_mask.txt` (include) or `variable_screen.txt` (exclude)

**Key Data Types:**
- Registry entries support various data types (USHORT, UINT, SHORT, INT, ASCII, HEX, bit fields)
- Dynamic register calculation with variable substitution and range expansion
- Multi-transport configurations for complex setups

### Protocol Development

When adding new protocols:
1. Create protocol JSON file in `protocols/{vendor}/` directory
2. Define registry maps in corresponding CSV files
3. Follow existing naming conventions for consistency
4. Test with actual hardware when possible

### Testing

The project uses pytest for testing. Tests are located in `pytests/` directory covering:
- Protocol settings parsing
- Transport configurations
- InfluxDB output formatting
- Example config validation