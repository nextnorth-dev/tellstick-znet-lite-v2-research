# Z-Wave Subsystem — Hardware, USB Interface and Firmware Architecture

[Documentation index](../README.md#documentation)

The examined firmware 1.3.2 unit has a dedicated Z-Wave controller exposed to
Linux as a USB CDC ACM serial device. The exact controller chip, firmware,
Serial API version, and third-party compatibility remain unidentified.

## Identified Hardware

The investigation identified a dedicated Sigma Designs Z-Wave controller connected to the Linux host.

| Component | Identification | Function |
| --- | --- | --- |
| Host processor | Atheros AR933x | Runs Linux and Telldus applications |
| Z-Wave controller | Sigma Designs | Z-Wave communication |
| USB vendor ID | 0658 | Sigma Designs |
| USB product ID | 0200 | Z-Wave controller interface |
| Linux driver | cdc_acm | USB serial communication |
| Linux device | /dev/ttyACM1 | Z-Wave serial interface |
| Firmware platform | OpenWrt / Linux 4.1.23 | Host operating system |

The exact Z-Wave controller chip model, hardware revision, firmware version and supported Z-Wave feature set have not been independently established.

The USB identification is not sufficient to determine all of those properties.

## System Architecture

The Z-Wave controller is part of the gateway’s internal USB architecture.

- TellStick ZNet Lite V2
  - Atheros AR933x
    - Linux 4.1.23
    - Telldus application
- |
  - Z-Wave software adapter
- |
  - Device management
- |
  - Application events
  - USB host controller
    - Internal USB hub
      - Sigma Designs Z-Wave controller
        - USB CDC ACM
        - /dev/ttyACM1
        - Z-Wave radio
  - Separate RF433 subsystem
    - PIC18F25K50
    - MAX7030
    - /dev/ttyACM0

This is a logical architecture outline based on the identified components and interfaces.

It is not a complete electrical schematic.

## USB Identification

The Z-Wave controller was identified with the following USB vendor and product identifiers:

```text
Vendor ID:  0658
Product ID: 0200
```

These identifiers are associated with Sigma Designs Z-Wave controller hardware.

The interface is exposed to Linux as:

```text
/dev/ttyACM1
```

### Original Kernel Log

The captured boot sequence includes:

```text
[   11.839356] cdc_acm 1-1.1:1.0: ttyACM0: USB ACM device
[   11.844816] cdc_acm 1-1.4:1.0: ttyACM1: USB ACM device
```

The two interfaces correspond to different internal controllers.

| Device | Subsystem |
| --- | --- |
| /dev/ttyACM0 | RF433 |
| /dev/ttyACM1 | Z-Wave |

The association was established on the examined device.

Linux device numbering is determined during enumeration and should not be assumed to remain identical under every hardware or boot configuration.

## USB CDC ACM Communication

The controller uses the Linux USB Communications Device Class Abstract Control Model driver.

The kernel module is commonly identified as:

```text
cdc_acm
```

This driver exposes compatible USB devices through character-device interfaces such as:

```text
/dev/ttyACM1
```

### Communication Layers

1. Telldus application
2. Z-Wave software adapter
3. Linux serial interface
4. USB CDC ACM
5. Sigma Designs controller
6. Z-Wave radio network

The USB connection carries communication between the host application and the dedicated controller.

It does not mean that the Linux host directly handles all radio timing, modulation, routing or protocol operations.

Those responsibilities may be divided between the controller firmware and host software.

The precise division has not been completely characterized.

## Z-Wave and RF433 Are Separate Systems

The TellStick ZNet Lite V2 supports two distinct radio subsystems.

Although both appear as USB serial devices, their hardware and protocol architectures differ.

| Property | Z-Wave | RF433 |
| --- | --- | --- |
| Controller manufacturer | Sigma Designs | Microchip |
| Identified controller | Exact model not established | PIC18F25K50 |
| Radio hardware | Dedicated Z-Wave controller | Maxim MAX7030 |
| USB ID | 0658:0200 | 1781:0c32 |
| Observed Linux device | /dev/ttyACM1 | /dev/ttyACM0 |
| Host interface | USB CDC ACM | USB CDC ACM |
| Protocol | Z-Wave | Supported 433 MHz protocols |
| Network behavior | Z-Wave protocol stack | Protocol-dependent |

These differences are important when considering driver support, firmware modification or independent integrations.

A technique applicable to the RF433 controller cannot automatically be applied to the Z-Wave controller.

## Software Integration

The stock Telldus firmware includes Python-based application components responsible for interacting with the radio subsystems.

The main application entry point is:

```text
/usr/bin/tellstick-znet.py
```

The application stack includes Z-Wave adapter functionality.

### Logical Application Flow

1. Application startup
2. Z-Wave adapter initialization
3. Serial communication
4. /dev/ttyACM1
5. Z-Wave controller
6. Z-Wave network

The application may use the controller for operations such as device communication and network management.

However, the precise commands, supported management functions and error-handling behavior have not been fully documented in this investigation.

### Startup Considerations

The production Telldus application uses explicit initialization of selected components.

The presence of a Python module or package entry point does not necessarily mean that arbitrary replacement modules will be loaded automatically.

Any independent integration would need to account for the actual application startup path and serial-port ownership.

## Controller Ownership and Serial Access

The stock Telldus application may maintain an active connection to the Z-Wave serial interface.

This matters when attempting independent inspection.

### Concurrent Access

Opening /dev/ttyACM1 from multiple applications can cause problems such as:

- Competing reads
- Unexpected consumption of controller responses
- Interrupted message sequences
- Application timeouts
- Conflicting controller operations

The interface should not be treated as a broadcast data stream that multiple applications can safely read independently.

### Safe Inspection Approach

For initial investigation, read-only system information is preferable to sending undocumented commands to the controller.

Commands that change Z-Wave network state, reset controller memory or modify inclusion information should be kept separate from basic interface identification.

## Read-Only Inspection Commands

The command blocks below are inspection examples, not captured session output.

The following commands can be used on a device with authorized Linux shell access.

### Confirm the Serial Device

```text
ls -l /dev/ttyACM1
```

### Inspect USB Enumeration

```text
dmesg | grep -i -E 'usb|cdc_acm|ttyACM'
```

### Inspect the Device’s Sysfs Information

```text
cat /sys/class/tty/ttyACM1/device/uevent
```

### Inspect USB Device Descriptors

If the utility is installed:

```text
lsusb
```

To inspect the identified controller:

```text
lsusb -d 0658:0200
```

### Inspect Driver Information

```text
ls -l /sys/class/tty/ttyACM1/device/driver
```

### Identify Processes Using the Interface

If supported by the installed userspace:

```text
fuser /dev/ttyACM1
```

These commands are intended for observation.

They do not deliberately send Z-Wave commands or modify controller state.

## Controller State and Network Data

Z-Wave controllers may retain network-specific information, depending on their hardware and firmware architecture.

Potentially sensitive data can include:

- Network identifiers
- Included-node information
- Controller configuration
- Security-related network material

The precise storage format and location on the examined controller have not been established.

Controller resets, exclusions, or network reinitialization can affect included
devices. No controller memory dump, security key, full network configuration,
or device-specific backup is published. Sanitize any protocol traces before
publication, especially for security-enabled networks.

## Findings and Limitations

| Research area | Finding | Status |
| --- | --- | --- |
| Dedicated Z-Wave controller | Present | Confirmed |
| Manufacturer identification | Sigma Designs | Confirmed |
| USB vendor ID | 0658 | Observed |
| USB product ID | 0200 | Observed |
| Linux interface | /dev/ttyACM1 | Confirmed |
| USB driver | cdc_acm | Confirmed |
| Internal USB connection | Kernel enumeration | Confirmed |
| Separate RF433 controller | Present | Confirmed |
| Telldus application integration | Z-Wave adapter components | Identified |
| Exact controller chip | Not established | Further research |
| Controller firmware version | Not established | Further research |
| Serial API version | Not established | Further research |
| Complete supported command set | Not established | Further research |
| Controller backup support | Not established | Further research |
| Third-party software compatibility | Not tested | Unknown |
| Controller firmware replacement | Not tested | Unknown |
| Complete electrical schematic | Not available | Further research |

## Related Documentation

- [Hardware Architecture](hardware.md)
- [Firmware and Filesystem Analysis](firmware.md)
- [UART Console and U-Boot](uart-and-uboot.md)
- [RF433 Subsystem](rf433.md)
- [Undocumented Wi-Fi](undocumented-wifi.md)
