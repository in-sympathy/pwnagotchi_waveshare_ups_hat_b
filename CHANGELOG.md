# Changelog

## 1.0.0

Initial release of the two-cell fork, derived from
[pokermaster1/waveshareupsbattery](https://github.com/pokermaster1/waveshareupsbattery).

- Target the **Waveshare UPS HAT** (original) and **UPS HAT (B)**, which use
  2× 18650 cells in series with an INA219 at I²C address **0x42**.
- Changed the INA219 address default from `0x43` to `0x42`.
- Replaced the single-cell percentage formula (`(V - 3.0) * 100 / 1.2`) with a
  two-cell one (`(V - 6.0) * 100 / 2.4`), exposed as `V_EMPTY` / `V_FULL`
  constants for easy calibration.
- Switched the INA219 calibration to the standard 32V/2A profile used by
  Waveshare's own UPS HAT demo code.
- Minor fix: corrected an undefined variable in the `on_unload` error handler.
- Namespaced log messages with `[wavesharebattery]`.
