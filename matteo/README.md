# Matteo's working setup (snapshot 2026-10-31)

Firmware running on three ESP8266 boards on a Toshiba multi-split (RAS-3M26U2AVG-E outdoor,
RAS-M07/M10/M22U2DVG-E ducted indoor units, RB-RWS21-E wired remotes, TU2C protocol).

This branch is a backup of the local component (`components/toshiba_ab`), based on the
`tu2c-room-temp-follower` branch of PR #256 plus local changes listed in `LOCAL_CHANGES.txt`.
It is not meant to be merged.

Bus setup on each indoor unit:
- the board is the **header** remote (`remote: 0x50`) and reports the room temperature
  (Zigbee sensor from Home Assistant) every 30 s with `61 38 01`;
- the wall remote is set as **follower** (MENU > Initial setting > 6. Header/Follower);
- FC 32 = 0001 on the unit (remote controller sensor).

Notes from testing:
- Reporting the room temperature from the follower address (0x51) next to a header wall
  remote works, but caused intermittent check code 30/32 (the wall remote sets byte 8 = 0x30
  in its `61 38` frame, the unit raises 32 and stops the indoor fan for ~3 min).
- As header, the room temperature must never stop: after ~3 min without `61 38` from 0x50
  the unit raises check code 30, stops the indoor fan and the compressor for that zone.
  The local firmware falls back to TA (−2 °C in heat, like FC 06) and reports right after
  registration.
- The unit answers every header `61 38` with a status frame. Reports colliding with the unit's
  own 5 s keepalive, or simply not answered (seen on one unit, ~1 in 3), can add up to 3 min
  without an accepted report → check code 30 for a few seconds. The local firmware does not
  transmit from 4.5 s after the unit's keepalive until the next one, and repeats an unanswered
  report after 3 s (up to 5 times).

The YAML files need a `secrets.yaml` next to them (`wifi_ssid`, `wifi_password`,
`fallback_hotspot_password`).
