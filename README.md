# Waveshare UPS HAT Battery Plugin for Pwnagotchi

A [Pwnagotchi](https://pwnagotchi.org/) plugin that shows the battery
percentage of the **Waveshare UPS HAT** (original) and **UPS HAT (B)** on the
Pwnagotchi display.

These two HATs use **two 18650 cells in series** (8.4 V charger) with an onboard
**INA219** current/voltage monitor on the I²C bus at address **0x42**. This
plugin reads the pack voltage over I²C and estimates the remaining charge.

> This is a fork of [pokermaster1/waveshareupsbattery](https://github.com/pokermaster1/waveshareupsbattery),
> which targets the single-cell **UPS HAT (C)** (INA219 @ 0x43). The changes
> here are the I²C address (0x43 → 0x42) and a two-cell percentage formula
> (6.0–8.4 V instead of 3.0–4.2 V), plus a calibration profile matching
> Waveshare's own UPS HAT demo code.

![UPS percentage on the Pwnagotchi screen](docs/screenshot.png)

## Supported hardware

| HAT | Cells | Charger | INA219 address | Works with this plugin |
|-----|-------|---------|----------------|------------------------|
| Waveshare UPS HAT (original) | 2× 18650 (series) | 8.4 V | 0x42 | ✅ yes |
| Waveshare UPS HAT (B)        | 2× 18650 (series) | 8.4 V | 0x42 | ✅ yes |
| Waveshare UPS HAT (C)        | 1× 18650          | 4.2 V | 0x43 | ❌ use the original [pokermaster1 plugin](https://github.com/pokermaster1/waveshareupsbattery) |

Confirm your HAT's address before installing:

```bash
sudo i2cdetect -y 1
```

You should see a device at **42**. If you see **43** instead, you have the
UPS HAT (C) — use the upstream plugin, not this one.

## Requirements

- A Pwnagotchi (tested on a Raspberry Pi Zero 2 W with the jayofelony image).
- I²C enabled (`sudo raspi-config` → Interface Options → I²C → enable, reboot).
- The `smbus` Python module (present on standard Pwnagotchi images). If it's
  missing: `sudo apt install python3-smbus`.

## Installation

1. Copy `wavesharebattery.py` into your custom plugins directory. On most
   builds that is:

   ```bash
   sudo cp wavesharebattery.py /etc/pwnagotchi/custom-plugins/
   ```

   (Check the `custom_plugins` path in your `config.toml` if unsure.)

2. Enable the plugin in `/etc/pwnagotchi/config.toml`:

   ```toml
   [main.plugins.wavesharebattery]
   enabled = true
   ```

3. Restart Pwnagotchi:

   ```bash
   sudo systemctl restart pwnagotchi
   ```

4. Confirm it loaded:

   ```bash
   grep -i wavesharebattery /etc/pwnagotchi/log/pwnagotchi.log
   ```

   You should see `[wavesharebattery] plugin loaded.` and a `UPS xx%` readout
   on the display.

## Verify your reading (optional)

Before trusting the percentage, confirm the pack voltage reads in the expected
2-cell range (~6.0–8.4 V):

```bash
python3 - <<'EOF'
import smbus
b = smbus.SMBus(1); addr = 0x42
raw = b.read_i2c_block_data(addr, 0x02, 2)
v = ((raw[0]*256 + raw[1]) >> 3) * 0.004
print("Bus voltage:", round(v, 3), "V")
print("Est %:", round(max(0, min(100, (v - 6.0) * 100 / 2.4))))
EOF
```

If it prints something near 3–4 V instead, your HAT is single-cell and you
want the UPS HAT (C) plugin.

## Calibration

The percentage is estimated from pack voltage, mapping **6.0 V → 0%** and
**8.4 V → 100%**. These are set near the top of the `WaveshareBattery` class:

```python
V_EMPTY = 6.0
V_FULL = 8.4
```

Waveshare notes that on these HATs the voltage-based estimate often tops out
slightly below 100% even when fully charged. If yours never reaches 100%,
lower `V_FULL` a little (e.g. `8.2`). If it sits at 100% too long, raise
`V_EMPTY` or `V_FULL` to taste.

## Known limitations

- **It's a voltage estimate, not a true fuel gauge.** The reading jumps around:
  it reads high while charging (the charger pushes the voltage up) and sags
  under load (when the Wi-Fi radio transmits). It's good for "roughly how full,"
  not for a precise gauge. For accuracy, a coulomb-counting gauge (e.g.
  MAX17043) would be the hardware upgrade.
- The `CHG`/`LOW` indicators key off the percentage, not actual charge current,
  and only re-evaluate every 100 UI updates. They are cosmetic. (True charge
  detection is possible via the INA219 current sign — positive = charging per
  Waveshare — and is a reasonable future enhancement.)

## Credits

- Original plugin and INA219 integration:
  [pokermaster1/waveshareupsbattery](https://github.com/pokermaster1/waveshareupsbattery)
- INA219 driver pattern: Waveshare UPS HAT example code.
- 2-cell fork: Oleksandr Golubets.

## License

GPL-3.0, inherited from the upstream plugin. See [LICENSE](LICENSE).
