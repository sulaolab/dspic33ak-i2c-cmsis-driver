# dspic33ak-i2c-cmsis-driver

CMSIS-Driver I2C wrapper package for the dsPIC33AK I2C HAL.

This repository is intended to provide a CMSIS-Driver I2C wrapper together with a vendor copy of the dsPIC33AK I2C HAL.

## Repository Layout

```text
src/
  hal_i2c/
    dspic33ak_i2c.h          (shared types + lifecycle)
    dspic33ak_i2c_master.h   dspic33ak_i2c_master.c
    dspic33ak_i2c_slave.h    dspic33ak_i2c_slave.c
    dspic33ak_i2c_common.h   dspic33ak_i2c_common.c
    dspic33ak_i2c_device.h   dspic33ak_i2c_device.c
    dspic33ak_i2c_reg.h
    UPSTREAM.md

tools/
  sync_hal_from_upstream.py

cmsis_driver/
  Driver_I2C_dsPIC33AK.c
  Driver_I2C_dsPIC33AK.h
  RTE_Device_I2C_dsPIC33AK_example.h
  README.md

third_party/
  arm_cmsis_driver/
    README.md
    LICENSE.txt
    Include/
      Driver_Common.h
      Driver_I2C.h
```

## Current Status

The HAL vendor copy has been imported from:

- https://github.com/sulaolab/dspic33ak-i2c-hal

The CMSIS-Driver wrapper files are provided under `cmsis_driver/`.

`RTE_Device_I2C_dsPIC33AK_example.h` is an I2C-only example configuration file.
Integrated applications should copy the required definitions into their own
application-level `RTE_Device.h` or equivalent configuration header.

A minimal copy of the ARM CMSIS-Driver API headers (`Driver_I2C.h`,
`Driver_Common.h`) is vendored under `third_party/arm_cmsis_driver/Include/` so
the wrapper builds without a separate CMSIS installation. See
`third_party/arm_cmsis_driver/README.md` for the source and license.

## Include Path

Applications or build systems should provide include paths for:

```text
src/hal_i2c
cmsis_driver
third_party/arm_cmsis_driver/Include
```

`Driver_I2C.h` is resolved from the vendored ARM CMSIS-Driver headers under
`third_party/arm_cmsis_driver/Include/` (Apache-2.0, copied unmodified). A
different CMSIS-Driver package may be substituted by adjusting this include path.

## HAL Synchronization

The HAL vendor copy under `src/hal_i2c/` can be synchronized from the upstream HAL-only repository using:

```powershell
# Windows (Python launcher)
py -3 tools/sync_hal_from_upstream.py

# or, if python is on PATH
python tools/sync_hal_from_upstream.py
```

## Upstream HAL Policy

The HAL-only repository is the upstream source of truth:

- https://github.com/sulaolab/dspic33ak-i2c-hal

HAL fixes should be applied to the upstream HAL repository first, then synchronized into this repository.

CMSIS-Driver wrapper changes should be made in this repository.

## License

The original dsPIC33AK I2C CMSIS-Driver wrapper code in this repository is
licensed under the MIT No Attribution License (MIT-0). See `LICENSE`.

The vendored ARM CMSIS-Driver headers under
`third_party/arm_cmsis_driver/` are provided by ARM under the Apache License
2.0. See `third_party/arm_cmsis_driver/LICENSE.txt`.
