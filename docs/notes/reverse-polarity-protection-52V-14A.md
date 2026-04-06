# 52 V / 14 A Reverse-Polarity Protection Sizing

## Design target

- Input supply: `52 VDC nominal`
- Continuous load current: `14 A`
- Goal: protect the input against reverse supply connection with low loss and low voltage drop
- Context: this is a good fit for a DRV8432 power rail, whose local datasheet shows a recommended supply up to `52.5 V`

## Shared-input case for two DRV8432 chips

If both DRV8432 chips share the same `52 V` input rail, you normally use:

- `one common reverse-polarity protection stage`
- placed at the main board power entry
- sized for the total current of both branches

If each branch can draw up to `14 A`, size the shared stage for:

- `28 A continuous`

For this case, the simple single `P-channel MOSFET` approach is no longer attractive.

Recommended topology for `28 A`:

- minimum: `2 back-to-back N-channel MOSFETs`
- preferred: `4 N-channel MOSFETs total`, arranged as `two back-to-back branches in parallel`
- use an `ideal-diode / reverse-polarity controller`

In other words:

- `2 MOSFETs` = required for the reverse-polarity function
- `4 MOSFETs` = often the better answer for `28 A continuous` because it cuts conduction loss roughly in half

Important note:

- the recommendation above is for a generic reverse-polarity stage
- if you use `LTC4359`, the controller can do reverse-input protection with `one N-channel MOSFET`
- with `LTC4359`, a `second MOSFET` is only needed when you also want `load switching / full disconnect`

## Recommended topology

For `52 V / 14 A`, the best practical approach is:

- `High-side reverse-polarity protection using N-channel MOSFETs`
- Prefer `back-to-back MOSFETs` if you want strong blocking and a robust design
- Use a `dedicated ideal-diode / reverse-polarity controller` or an equivalent high-side gate-drive arrangement

This is strongly preferred over a series diode because the power loss is much lower.

## Why a diode is usually a bad fit here

At `14 A`, a series diode wastes a lot of power:

- Schottky example: `0.55 V * 14 A = 7.7 W`
- Silicon diode example: `0.9 V * 14 A = 12.6 W`
- Bridge rectifier is even worse because two diode drops are in the path

That amount of heat is usually not worth it unless you are intentionally trading efficiency for simplicity.

## Resistance target

For a MOSFET-based solution, size the total series resistance from the power loss you can tolerate.

Formula:

```text
P = I^2 * R
R = P / I^2
```

At `14 A`:

- If target loss is `2 W total`:

```text
R_total = 2 / (14^2) = 2 / 196 = 0.0102 ohm
R_total = 10.2 mOhm
```

- Voltage drop at that resistance:

```text
V_drop = I * R = 14 * 0.0102 = 0.143 V
```

So a solid design target is:

- `Total hot resistance <= 10 mOhm`

### Resistance target for `28 A`

At `28 A`, losses rise fast because they scale with `I^2`.

```text
I^2 = 28^2 = 784
```

Useful targets:

```text
For 2 W total loss:
R_total = 2 / 784 = 0.00255 ohm = 2.55 mOhm

For 3 W total loss:
R_total = 3 / 784 = 0.00383 ohm = 3.83 mOhm

For 4 W total loss:
R_total = 4 / 784 = 0.00510 ohm = 5.10 mOhm
```

This is why `28 A` usually pushes the design toward multiple N-MOSFETs instead of a single device.

## MOSFET sizing target

### If using one series MOSFET

Target:

- `Rds(on)_hot <= 10 mOhm total`

Because MOSFET resistance rises with temperature, room-temperature resistance should usually be lower. A practical target is:

- `Rds(on)_25C <= 6 mOhm`

### If using two back-to-back MOSFETs

The total resistance is the sum of both devices:

- `R_total = Rds(on)_FET1 + Rds(on)_FET2`

For the same `10 mOhm` hot target:

- `Each MOSFET should be <= 5 mOhm hot`
- Practical room-temperature target: `<= 3 mOhm per MOSFET at 25 C`

Example:

```text
Each FET hot resistance = 5 mOhm
Total resistance = 10 mOhm
Total loss = 14^2 * 0.010 = 1.96 W
Loss per FET = 0.98 W
```

That is a much easier thermal problem than a diode solution.

## Voltage rating

For a `52 V` rail, do not choose a MOSFET right at the rail voltage.

Recommended minimums:

- `MOSFET Vds >= 80 V`
- `100 V preferred` if there is cable length, hot-plugging, motor noise, or supply overshoot

The same applies to the reverse-polarity controller and any gate clamp components.

## LTC4359-specific MOSFET sizing

If you use `LTC4359IS8`, the MOSFET sizing rules come from the local `ltc4359.pdf`:

- it controls a `single N-channel MOSFET` for ideal-diode and reverse-input protection
- it can also control `back-to-back MOSFETs`, but that is mainly for `load switching / full disconnect`
- the datasheet says `100 mV` forward drop is a good starting point
- the datasheet gives:

```text
Rds(on) < Forward Voltage Drop / ILOAD
Pd = ILOAD^2 * Rds(on)
```

### For your `28 A` case

Using the LTC4359 datasheet's `100 mV` starting point:

```text
Rds(on)_total < 0.1 / 28 = 0.00357 ohm
Rds(on)_total < 3.57 mOhm
```

So the clean starting target is:

- `single-MOSFET solution: <= 3.6 mOhm total at operating temperature`

That means in practice:

- aim for about `<= 2 mOhm to 2.5 mOhm at 25 C`
- because `Rds(on)` rises when the MOSFET gets hot

### Power loss examples for a single MOSFET at `28 A`

```text
At 2.0 mOhm:
P = 28^2 * 0.0020 = 1.57 W
Vdrop = 28 * 0.0020 = 56 mV

At 2.6 mOhm:
P = 28^2 * 0.0026 = 2.04 W
Vdrop = 28 * 0.0026 = 73 mV

At 3.6 mOhm:
P = 28^2 * 0.00357 = 2.80 W
Vdrop = 28 * 0.00357 = 100 mV
```

So for `LTC4359 + 28 A`, a single low-Rds(on) N-MOSFET can be completely reasonable if the package and thermal path are strong enough.

### When to use two MOSFETs with LTC4359

Use `2 back-to-back MOSFETs` only if you need one of these:

- full output disconnect when `SHDN` is low
- load-switch behavior
- inrush control using the datasheet's load-switch arrangement

If you only want `reverse-input protection` and `low-loss ideal-diode behavior`, `1 MOSFET` is enough with LTC4359.

### Critical voltage-rating warning at `52 V`

The same `ltc4359.pdf` also says:

- `BVDSS` must be higher than the supply voltage
- if the input is reversed while the output is held up, the MOSFET can see `OUT + |VIN|`

For a nominal `52 V` system:

```text
Worst-case reverse event with output held up:
Vds > 52 V + 52 V = 104 V
```

That means:

- `100 V MOSFETs are not comfortable` for a true full reverse connection case
- `150 V MOSFETs` are the safer class if you really want to survive a full `-52 V` reverse event with output hold-up

Also important:

- the LTC4359 pins themselves are only rated to `-40 V` directly
- so a `52 V` reverse battery event requires the external clamp / diode network to be designed like the higher-voltage examples in the datasheet
- do not expose the LTC4359 pins directly to `-52 V`

### Practical recommendation with LTC4359 at `52 V / 28 A`

- use `1 x LTC4359IS8`
- use `1 x N-channel MOSFET` if you only need reverse-input protection
- target about `<= 2 mOhm to 2.5 mOhm at 25 C`
- prefer a strong package such as `TO-220`, `TO-247`, `D2PAK`, `TO-263`, `LFPAK`, or similar
- use a `12 V` gate-source zener clamp as shown in the LTC4359 datasheet examples for `24 V and higher`
- if you need full disconnect or controlled turn-on, move to `2 back-to-back MOSFETs`

### Selected MOSFET

Recommended part:

- `XRS200N15TL`

Why this is the pick:

- `150 V` rating gives the right margin class for a `52 V` rail with reverse-input events
- `Rds(on) = 3.2 mOhm @ 10 V` from the local datasheet
- `TOLL-8L` is a strong SMD power package
- local datasheet also shows `RthJC = 0.36 C/W`
- this is a reasonable fit for the `LTC4359 + 52 V / 28 A` target even though it is not the absolute lowest-loss option

What that means at `28 A`:

```text
Vdrop = 28 * 0.0032 = 0.0896 V
P = 28^2 * 0.0032 = 2.51 W
```

So this part still meets the LTC4359 sizing target based on the local datasheet.

Practical use:

- if you only need reverse-input protection and ideal-diode behavior, use `1 x XRS200N15TL`
- if you also want full disconnect when shutdown is asserted, use `2 x XRS200N15TL` in the LTC4359 back-to-back arrangement

Lower-loss alternative:

- `Infineon IAUTN15S6N025ATMA1`

That part has lower loss:

```text
Rds(on) max = 2.5 mOhm
Vdrop at 28 A = 0.070 V
P at 28 A = 1.96 W
```

But since you chose `XRS200N15TL`, it is acceptable to proceed with it.

## Current rating

Do not size from the headline `Id` rating alone.

Instead, check:

- `Rds(on)` at temperature
- package thermal performance
- copper area on the PCB
- connector and wire current capability

At around `1 W per MOSFET`, common power packages can work well with proper copper:

- `TO-263 / D2PAK`
- `PowerSO-8`
- `LFPAK`
- similar low-thermal-resistance packages

Very small packages usually need either:

- more copper area, or
- parallel MOSFETs

## If you want the simplest circuit

A `high-side P-channel MOSFET` can do reverse-polarity protection with fewer parts, but it is usually less attractive at `52 V / 14 A`.

Example losses:

- `20 mOhm`: `14^2 * 0.020 = 3.92 W`
- `30 mOhm`: `14^2 * 0.030 = 5.88 W`

This can still work, but it runs hotter and low-Rds(on) high-voltage P-MOSFETs are harder to find.

Important:

- add a `Vgs clamp zener` because many MOSFETs only allow about `+/-20 V` gate-to-source
- at `52 V`, directly pulling the gate to ground without a clamp can destroy the MOSFET

## Practical starting point

Preferred solution:

- `1 high-side reverse-polarity / ideal-diode controller`
- `2 back-to-back N-channel MOSFETs`
- `80 V to 100 V` MOSFET rating
- aim for `<= 3 mOhm at 25 C` per MOSFET

Expected result:

- about `2 W total loss` at `14 A` if the pair is around `10 mOhm hot`
- only about `0.14 V` drop at full current

## Example component suggestions

### Simpler MOSFET + diode version

If you want the simplest practical implementation with a small part count, use:

- `Q1: Vishay SQM120P10-10m1L`
- `D1: Nexperia NZH15B`
- optional `D2: Littelfuse SMDJ58A`

What each part does:

- `Q1` is the main high-side reverse-polarity MOSFET
- `D1` is a `15 V` zener used as a `gate-source clamp`
- `D2` is an input TVS for surge and hot-plug protection

Key fit for this application:

- `SQM120P10-10m1L`: `100 V` P-channel MOSFET, `Rds(on) max 10.1 mOhm at Vgs = -10 V`
- `NZH15B`: about `15 V` zener clamp level, suitable for protecting the MOSFET `Vgs`
- `SMDJ58A`: `58 V` standoff TVS, `3 kW` surge rating

Estimated Q1 conduction loss at `14 A` from the local `SQM120P10 datasheet.pdf`:

```text
At 25 C, max Rds(on) = 10.1 mOhm:
P = 14^2 * 0.0101 = 1.98 W

At 125 C, max Rds(on) = 16.8 mOhm:
P = 14^2 * 0.0168 = 3.29 W

At 175 C, max Rds(on) = 20.5 mOhm:
P = 14^2 * 0.0205 = 4.02 W
```

Associated voltage drop at `14 A`:

```text
At 25 C,  Vdrop = 14 * 0.0101 = 0.141 V
At 125 C, Vdrop = 14 * 0.0168 = 0.235 V
At 175 C, Vdrop = 14 * 0.0205 = 0.287 V
```

Important correction:

- the earlier `~1.98 W` figure is only the `25 C` conduction-loss estimate
- for continuous `14 A`, design closer to the hot numbers, not only the room-temperature number
- this makes the single P-MOSFET option much less attractive thermally unless you have a strong thermal path and plenty of copper

Thermal warning from the same datasheet:

- `RthJA = 40 C/W` on a `1 inch square PCB` test condition
- even `3.29 W` would imply a very large junction rise in a poor thermal setup
- in practice, do not treat the single P-MOSFET option as a carefree `14 A continuous` solution

### Wiring note for the simple P-MOSFET version

- `Q1 source -> raw +52 V input`
- `Q1 drain -> protected +52 V rail`
- `Q1 gate -> GND through about 100 kOhm`
- `D1 zener between gate and source`
- `D1 cathode -> source`
- `D1 anode -> gate`
- optional `47 to 100 ohm` series gate resistor
- optional `D2 TVS` from protected rail to GND

Important:

- do not pull the P-MOSFET gate directly far below the source without a clamp
- many MOSFETs are only rated for `+/-20 V` on `Vgs`
- at `52 V`, the `15 V` zener is what keeps the gate oxide safe

### Lower-loss upgrade path

If thermal rise is a concern, use:

- `2 x 100 V N-channel MOSFETs` in back-to-back arrangement
- `1 x ideal-diode / reverse-polarity controller`

A good example MOSFET for that approach is:

- `Infineon IPP026N10NF2S`, `100 V`, `Rds(on) max 2.6 mOhm`

This approach gives much lower loss than the single P-MOSFET solution.

### What to do for `28 A`

For `28 A continuous`, a useful way to think about it is:

- `2 N-MOSFETs back-to-back` is the minimum functional solution
- `4 N-MOSFETs total` is often the better thermal solution

Worked example using a `100 V` N-MOSFET around `2.6 mOhm max at 25 C`:

#### Option A: `2 MOSFETs total` in back-to-back series

```text
R_total = 2.6 mOhm + 2.6 mOhm = 5.2 mOhm
P_total = 28^2 * 0.0052 = 4.08 W
Vdrop = 28 * 0.0052 = 0.146 V
```

This can work, but the actual hot loss will be higher than `4.08 W`.

#### Option B: `4 MOSFETs total` as two back-to-back branches in parallel

Each branch:

```text
R_branch = 2.6 mOhm + 2.6 mOhm = 5.2 mOhm
```

Two equal branches in parallel:

```text
R_total = 5.2 mOhm / 2 = 2.6 mOhm
P_total = 28^2 * 0.0026 = 2.04 W
Vdrop = 28 * 0.0026 = 0.073 V
```

This is why `4 MOSFETs` is usually the more comfortable answer for `28 A continuous`.

### Practical recommendation for `28 A`

- use `one shared reverse-polarity stage` at the board input
- use `4 x 100 V N-channel MOSFETs`
- arrange them as `2 series for reverse blocking`, with `2 such series branches in parallel`
- drive them with an `ideal-diode / reverse-polarity controller`
- keep the layout symmetrical so both parallel branches share current well
- if paralleled, give each MOSFET its own small gate resistor, for example `5 ohm to 22 ohm`

## Supporting protection

Reverse-polarity blocking is only one part of input protection. A robust input stage often also includes:

- input fuse or breaker sized to the cable and fault energy
- TVS diode if the cable is long or hot-plugging is expected
- local bulk capacitance after the protection stage
- short, wide copper on the high-current path

## Quick checklist

- `52 V / 14 A` is too much for a casual diode solution unless you accept heavy heating
- use `N-MOSFET high-side protection` if efficiency matters
- target `<= 10 mOhm total hot resistance`
- use `80 V minimum`, `100 V preferred`
- include a proper `gate clamp / controller`
- verify thermal rise at full current, not just room-temperature resistance

## Notes

- This note sizes the protection stage from the electrical requirement only
- TVS sizing, fuse value, and exact MOSFET selection still depend on your connector type, cable length, ambient temperature, and surge environment
- For the DRV8432 specifically, keep the downstream rail inside the device supply limits from the local datasheet: see `DRV8432DKDR.pdf`
