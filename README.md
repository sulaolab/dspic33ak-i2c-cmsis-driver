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
```

## Current Status

The HAL vendor copy has been imported from:

- https://github.com/sulaolab/dspic33ak-i2c-hal

The CMSIS-Driver wrapper files will be added in a later step.

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
