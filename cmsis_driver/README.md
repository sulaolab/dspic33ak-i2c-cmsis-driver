# CMSIS-Driver I2C Wrapper for dsPIC33AK I2C HAL

## Overview

This directory provides a CMSIS-Driver I2C wrapper for the dsPIC33AK blocking I2C HAL.

The current implementation is a blocking master-only driver intended to make the dsPIC33AK I2C HAL usable through the CMSIS-Driver I2C API.

## File layout

```text
cmsis_driver/
  Driver_I2C_dsPIC33AK.c
  Driver_I2C_dsPIC33AK.h
  RTE_Device.h
  README.md
```

In this repository, these files are located under the top-level `cmsis_driver/` folder.

## Instance mapping

| CMSIS driver object | dsPIC33AK HAL instance |
|---|---|
| `Driver_I2C0` | `DSPIC33AK_I2C_INST_1` |
| `Driver_I2C1` | `DSPIC33AK_I2C_INST_2` |
| `Driver_I2C2` | `DSPIC33AK_I2C_INST_3` |
| `Driver_I2C3` | `DSPIC33AK_I2C_INST_4` |

## Supported features

- Blocking master transmit
- Blocking master receive
- 7-bit addressing
- `MasterTransmit(..., xfer_pending = false)`
- `MasterReceive(..., xfer_pending = false)`
- `MasterTransmit(..., xfer_pending = true)` followed by `MasterReceive(..., xfer_pending = false)`
- Bus speed selection before `PowerControl(ARM_POWER_FULL)`:
  - Standard mode: 100 kHz
  - Fast mode: 400 kHz
  - Fast-mode Plus: 1 MHz

## Unsupported features and limitations

The initial wrapper intentionally does not support:

- `MasterReceive(..., xfer_pending = true)`
- Slave/client mode:
  - `SlaveTransmit()`
  - `SlaveReceive()`
- 10-bit addressing
- General Call
- High-speed mode
- Bus clear
- Abort transfer
- Interrupt-driven asynchronous transfer
- Changing bus speed while powered

Unsupported functions return `ARM_DRIVER_ERROR_UNSUPPORTED`.

## Configuration through RTE_Device.h

`RTE_Device.h` selects which CMSIS I2C driver objects are enabled and provides default timing parameters.

Applications should edit these settings for the target I2C instance, clock, bus speed, and timeout policy.

Example:

```c
#define RTE_I2C1 1

#define RTE_I2C1_FCY_HZ             100000000u
#define RTE_I2C1_BUS_HZ             400000u
#define RTE_I2C1_TIMEOUT_MS         10u
#define RTE_I2C1_PENDING_TIMEOUT_MS 0u
```

`RTE_I2C1` enables `Driver_I2C1`, which maps to `DSPIC33AK_I2C_INST_2`.

## Tick provider

The HAL timeout mechanism requires a millisecond tick provider.

The wrapper provides a weak default implementation:

```c
uint32_t Driver_I2C_dsPIC33AK_GetMs(void)
{
    return 0u;
}
```

Applications should override this function when timeout support is required.

Example:

```c
uint32_t Driver_I2C_dsPIC33AK_GetMs(void)
{
    return GetTicks();
}
```

## Include path

The wrapper intentionally uses header names without hard-coded relative paths.

This repository does not vendor the official CMSIS headers. The build system should provide include paths for:

```text
src/hal_i2c
cmsis_driver
path/to/CMSIS/Driver/Include
```

`Driver_I2C.h` is expected to come from the official CMSIS-Driver package.

## Basic usage

```c
#include "Driver_I2C_dsPIC33AK.h"

extern ARM_DRIVER_I2C Driver_I2C1;

uint8_t reg = 0x00;
uint8_t rx[2];

Driver_I2C1.Initialize(NULL);
Driver_I2C1.PowerControl(ARM_POWER_FULL);

Driver_I2C1.MasterTransmit(0x1A, &reg, 1, true);
Driver_I2C1.MasterReceive(0x1A, rx, 2, false);
```

This sequence performs a typical register read:

```text
START + address write + register address
Repeated START + address read + data read + STOP
```

## Verified behavior

The wrapper has been verified on the perseus_512_96K project using the WM8904 codec register read path.

Verified sequence:

```c
Driver_I2Cx.MasterTransmit(addr7, &reg, 1, true);
Driver_I2Cx.MasterReceive(addr7, rx, 2, false);
```

The WM8904 device ID register was read successfully as `0x8904`.
