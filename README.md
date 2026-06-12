# dspic33ak-i2c-cmsis-driver

CMSIS-Driver I2C wrapper package for the dsPIC33AK I2C HAL.

This repository is intended to provide a CMSIS-Driver I2C wrapper together with a vendor copy of the dsPIC33AK I2C HAL.

## Repository Layout

```text
src/
  hal_i2c/
    dspic33ak_i2c.c
    dspic33ak_i2c.h
    dspic33ak_i2c_device.c
    dspic33ak_i2c_device.h
    dspic33ak_i2c_reg.h
    UPSTREAM.md

tools/
  sync_hal_from_upstream.py

cmsis_driver/
  Driver_I2C_dsPIC33AK.c
  Driver_I2C_dsPIC33AK.h
  RTE_Device.h
  README.md
```

## Current Status

The HAL vendor copy has been imported from:

- https://github.com/sulaolab/dspic33ak-i2c-hal

The CMSIS-Driver wrapper files are provided under `cmsis_driver/`.

This repository does not vendor the official CMSIS headers.

## Include Path

Applications or build systems should provide include paths for:

```text
src/hal_i2c
cmsis_driver
path/to/CMSIS/Driver/Include
```

`Driver_I2C.h` is expected to come from the official CMSIS-Driver package.

## HAL Synchronization

The HAL vendor copy under `src/hal_i2c/` can be synchronized from the upstream HAL-only repository using:

```powershell
python tools/sync_hal_from_upstream.py
```

## Upstream HAL Policy

The HAL-only repository is the upstream source of truth:

- https://github.com/sulaolab/dspic33ak-i2c-hal

HAL fixes should be applied to the upstream HAL repository first, then synchronized into this repository.

CMSIS-Driver wrapper changes should be made in this repository.
