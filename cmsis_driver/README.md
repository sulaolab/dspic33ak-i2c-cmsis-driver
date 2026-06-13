# CMSIS-Driver I2C Wrapper for dsPIC33AK I2C HAL

## Overview

This directory provides a CMSIS-Driver I2C wrapper for the dsPIC33AK blocking I2C HAL.

The current implementation is a blocking master-only driver intended to make the dsPIC33AK I2C HAL usable through the CMSIS-Driver I2C API.

## File layout

```text
cmsis_driver/
  Driver_I2C_dsPIC33AK.c
  Driver_I2C_dsPIC33AK.h
  RTE_Device_I2C_dsPIC33AK_example.h
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
- Bus speed selection via `Control(ARM_I2C_BUS_SPEED, ...)`, callable before or
  after `PowerControl(ARM_POWER_FULL)` (after power-on the bus must be idle):
  - Standard mode: 100 kHz
  - Fast mode: 400 kHz
  - Fast-mode Plus: 1 MHz
  - High-speed mode (`ARM_I2C_BUS_SPEED_HIGH`): unsupported
  - Returns `ARM_DRIVER_ERROR_BUSY` during an active transfer or a pending
    no-STOP transaction

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

Unsupported functions return `ARM_DRIVER_ERROR_UNSUPPORTED`.

## Configuration through RTE_Device_I2C_dsPIC33AK_example.h

`RTE_Device_I2C_dsPIC33AK_example.h` is an example configuration file for this I2C
CMSIS-Driver wrapper. It is not intended to be a shared application-level
`RTE_Device.h`; in an integrated application, copy the required I2C definitions
into that application's own `RTE_Device.h` or equivalent configuration header. It
selects which CMSIS I2C driver objects are enabled and provides default timing
parameters.

The bundled sample enables `RTE_I2C1` and `RTE_I2C2` (and leaves `RTE_I2C0` and `RTE_I2C3` disabled) only as an example. Applications should edit the `RTE_I2Cx` enable flags and timing values (clock, bus speed, and timeout policy) to match the target board's I2C instance and clock configuration.

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

The dsPIC33AK I2C HAL vendor copy is provided in this repository under `src/hal_i2c/`. A minimal copy of the ARM CMSIS-Driver API headers is vendored under `third_party/arm_cmsis_driver/Include/`. The build system should provide include paths for:

```text
src/hal_i2c
cmsis_driver
third_party/arm_cmsis_driver/Include
```

`Driver_I2C.h` is resolved from the vendored ARM CMSIS-Driver headers under `third_party/arm_cmsis_driver/Include/` (Apache-2.0, copied unmodified). A different CMSIS-Driver package may be substituted by adjusting this include path.

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

Runtime bus speed change was also verified on the same project:

```c
Driver_I2Cx.Initialize(NULL);
Driver_I2Cx.PowerControl(ARM_POWER_FULL);
Driver_I2Cx.Control(ARM_I2C_BUS_SPEED, ARM_I2C_BUS_SPEED_STANDARD);
```

`Control(ARM_I2C_BUS_SPEED, ...)` after `PowerControl(ARM_POWER_FULL)` returned
`ARM_DRIVER_OK` and the WM8904 device ID read / init succeeded at the new speed.
Verified at 100 kHz, 150 kHz, 200 kHz, 300 kHz and 400 kHz. The 100 kHz case in
particular depends on the upstream HAL STOP-completion fix (STOP waits for
`CON1.PEN` to clear, not just `STAT2.STOPE`).
