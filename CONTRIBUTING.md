# Contributing

Thanks for helping keep the Polaroid SX-70 alive. This is a community effort — contributions of all
sizes are welcome, from a typo fix to a full bench-test report.

> **License note:** this project is **CC BY-NC-SA 4.0**. By contributing you agree your contribution
> is released under the same license, and that your own findings/derivatives stay **open** and
> **non-commercial** (see `LICENSE`). Please don't submit anything containing Polaroid firmware,
> software, or proprietary data — this project deliberately contains none.

## Where help is most valuable
1. **Measurements of real film packs and cameras.** The outline, the position and size of the
   camera's battery contacts (and which one is + and which is −), the space available inside the
   film chamber, and where the film window sits. Calipers, photos with a scale, and meter readings
   are all useful.
2. **Camera current profiles.** The current an SX-70 draws during a cycle (motor start, running,
   stall, shutter solenoid) and how long each peak lasts. The fuse ratings, current limit and cell
   choice depend on it.
3. **Cell suggestions.** Thin (≤ about 8 mm), single-cell lithium pouch cells around 20 × 50 mm, with
   built-in protection and a pulse rating that covers the camera's peaks.
4. **Firmware.** There is no working ESP32-S3 firmware release yet: sensor timing capture, brightness
   readout, camera power control, measurements and a wireless interface. The pin map is in the
   [hardware reference](docs/HARDWARE.md).
5. **Design review.** The issues in the [validation record](docs/VALIDATION.md), or anything else you
   spot.

## How to contribute
1. Fork the repo and create a branch.
2. Make your change. For fixes to the production files, describe exactly what changed and why.
3. Validate it: for design changes, include your DRC/ERC results; for measurements and test reports,
   include your setup (camera model, film pack, instruments), what you measured, and photos if you
   can.
4. Keep the README in step with any change to the files or to what has been verified.
5. Open a pull request describing **what** you changed, **why**, and **how you validated it**. In
   keeping with the license, please share the findings behind your change openly, not just the
   files.

## Reporting issues
Open an issue with the board revision, your camera model, what you expected and what happened, plus
measurements or photos where possible. Never attach private personal data or any Polaroid material.

## Code of conduct
Be kind and constructive. This is a small project for owners of long-discontinued cameras; assume
good faith.

## Support
If this helped you, support ongoing SX-70 work at **[ko-fi.com/oakman](https://ko-fi.com/oakman)**
— it funds hardware, test cameras, film, and time for further development.
