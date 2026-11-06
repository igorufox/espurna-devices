# ESPurna devices

Board templates for [ESPurna](https://github.com/igorufox/espurna) (`esp32-port` branch), for boards that are in neither
the [Tasmota templates](https://templates.blakadder.com) nor the [OpenBeken devices](https://openbekeniot.github.io/webapp/devicesList.html)
list, or that need ESPurna-only settings (dummy relays, wall switch modes, the 433 MHz receiver ...). This way no-name
boards do not have to be built into the firmware: flash the generic image of the chip, then pick the board.

The ESPurna WebUI reads `devices.json` straight from this repository, in the browser, at the time of the search:
*ADMIN* → *Device Template* → *Find device*, results marked `[github]`. *Apply* writes the settings and asks to reboot.
The device itself does not need internet access, and nothing is sent anywhere.

Default URL: `https://raw.githubusercontent.com/igorufox/espurna-devices/main/devices.json`.
Another list (a fork, a local web server with CORS) is set with the `webTplUrl` setting, *Own device list* in the same
panel, or `set webTplUrl <url>` in the terminal. Empty = the default one.

## Format

`devices.json` is a JSON array of entries:

```json
{
    "name": "Sparkleiot MINI 通断器 V1.3 (XH-CB2S, BK7231M)",
    "model": "MINI V1.3 XH-CB2S",
    "category": "relay",
    "arch": "beken",
    "mcu": "BK7231N",
    "image": "beken-matter",
    "link": "https://example.com/board-photos",
    "notes": "S1 is tied to N, flash it only without mains.",
    "settings": {
        "relayGpio0": "24",
        "relayType0": "normal",
        "sysManufacturer": "SPARKLEIOT",
        "sysDevice": "MINI_XH_CB2S"
    }
}
```

| Field | | |
|-------|---|---|
| `name` | required | shown in the results, searched |
| `model` | optional | shown and searched |
| `category` | optional | searched (`relay`, `switch`, `plug` ...) |
| `arch` | required | `esp8266`, `esp32` or `beken`, the entry is only listed on that chip |
| `mcu` | optional | the chip as the device reports it (`BK7231N`, `BK7231T` ...), the entry is only listed on that chip. Leave it out for ESP8266 / ESP32, ESP32 devices report the chip revision |
| `image` | optional | firmware image / build the board needs (e.g. `beken-matter`, `esp8266-1m`, `esp8266-1m-sensor`), shown in the results and in the *Apply* confirmation. The device cannot tell which image it runs, so it is not checked |
| `link` | optional | page of the board, the `[github]` tag becomes a link |
| `notes` | optional | shown in the *Apply* confirmation and as the tooltip of the result |
| `settings` | required | ESPurna settings, key → value, **values are strings** |

Indexes in the keys count from 0 (`relayGpio0` is the first relay, `btnRelay1: "0"` = the second button controls the first relay).

## Adding a board

1. Flash the generic image of the chip, set the board up by hand on the RELAYS / LED panels (or apply a similar template
   and change it), and check that every relay, button and LED works.
2. *ADMIN* → *Device Template* → *Export this device as a template* → **Export as template**. The WebUI reads the saved
   settings of the device (the same data as *Backup*) and keeps only the hardware keys; **Copy**.
3. Fill in `name`, `image` and `notes` (and `mcu` if it was not filled in), add the entry to `devices.json`, keep the file
   valid JSON (an array, commas between entries) and commit.

Settings at their default value are not stored on the device, so they are not exported either; that is fine as long as
the board uses the same image.

## Safety

- Only hardware keys are exported, and only hardware keys are ever applied from this list, whatever an entry contains:
  `relayGpio` `relayGpioType` `relayType` `relayProv` `relayResetGpio` `relayDummy`,
  `btnGpio` `btnGpioType` `btnProv` `btnMode` `btnPinMode` `btnDefVal` `btnRelay` `btnPress` `btnRlse` `btnClick` `btnDclk`
  `btnTclk` `btnLclk` `btnLLclk` `btnLclkDel` `btnLLclkDel` `btnDebDel` `btnRepDel` `btnFreq` `btnMinPulses` `btnMinDuty`
  `btnAcConfirm` `btnAcFreq`, `ledGpio` `ledInv` `ledMode` `ledRelay`, `ltDimmerGPIO` `ltEnableGPIO` `ltInv`,
  `uartTx` `uartRx` `uartBaud` `uartDataBits` `uartParity` `uartStopBits` `uartInv`, `rfbRX` `rfbTX`,
  `sysManufacturer` `sysDevice`. Wi-Fi, passwords, API key, MQTT, topics, relay sync, Home Assistant, NTP, telnet,
  hostname and description never get in or out.
- Still read the exported entry before committing it, the repository is public.
- A wrong pin can drive a relay or short an input: only add boards that were tested on real hardware with these exact settings.
- Many in-wall modules are not isolated from mains (e.g. a switch terminal tied to N). Write that in `notes`, and
  never connect a USB-UART adapter to a board powered from mains.
