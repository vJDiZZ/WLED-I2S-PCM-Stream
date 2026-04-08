# I2S Slave Mode for WLED AudioReactive

## Overview

The I2S Slave audio source allows the ESP32 to receive audio data from an **external I2S master device** such as a DSP (e.g., Analog Devices ADAU1701), DAC, or other audio processor that generates its own BCLK and LRCLK signals.

Unlike the standard I2S sources where the ESP32 acts as master and generates the clock signals, in slave mode the ESP32 listens for clocks from the external device.

## Configuration

In the WLED web interface:
1. Go to **Config** > **Sound Settings**
2. Set **Audio Source** to **I2S Slave** (type 10)
3. Configure the I2S pins:
   - **I2S SD** (Serial Data) — audio data input from master
   - **I2S SCK** (Serial Clock / BCLK) — bit clock from master
   - **I2S WS** (Word Select / LRCLK) — left/right clock from master

## Pin Wiring Example (ADAU1701 → ESP32-S3)

| ADAU1701 Pin | Signal | ESP32-S3 GPIO | WLED Config |
|--------------|--------|---------------|-------------|
| MP6 (SDATA_OUT0) | Serial Data | GPIO 10 | I2S SD |
| MP5 (BCLK_OUT0) | Bit Clock | GPIO 11 | I2S SCK |
| MP4 (LRCLK_OUT0) | Word Select | GPIO 12 | I2S WS |
| GND | Ground | GND | — |

## Build Configuration

Default pin assignments can be set at build time in `platformio.ini`:

```ini
-D SR_DMTYPE=10           ; I2S Slave source type
-D I2S_SDPIN=10           ; Serial Data pin
-D I2S_CKPIN=11           ; Bit Clock pin
-D I2S_WSPIN=12           ; Word Select pin
-D MCLK_PIN=-1            ; No master clock (slave mode)
```

## Technical Details

- **Sample format**: 32-bit stereo, left-justified (standard I2S)
- **Channel**: Left channel is extracted from interleaved stereo data
- **Normalization**: 32-bit samples are scaled to 16-bit range (`/65536.0f`) matching the `I2S_SAMPLE_DOWNSCALE_TO_16BIT` convention used by all standard I2S sources
- **Timeout**: 100ms read timeout prevents blocking when no master clocks are present
- **Pin mode**: All I2S pins (WS, SD, SCK) are configured as high-impedance inputs before driver initialization to prevent bus interference
- **No APLL**: The audio PLL is not used in slave mode since the master controls timing

## Supported Platforms

- ESP32 (classic)
- ESP32-S3

Not supported on ESP32-S2 or ESP32-C3 (I2S slave mode limitations).
