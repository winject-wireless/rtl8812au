# Monitor-mode receive gain override

In monitor mode the stock dynamic initial gain (DIG) has no associated peer to
anchor on and parks the IGI at the bottom of its range (0x1c-0x20). On a quiet
channel this produces thousands of false alarms per second and a constant
10-20% frame loss, even at good signal level.

Two module parameters, writable at runtime, pin the values instead:

| Parameter | Range | Meaning |
|---|---|---|
| `rtw_monitor_igi` | `0`, `0x1c`-`0x5a` | Fixed IGI (about 1 dB per step, RSSI + 110 convention). `0` = stock DIG. |
| `rtw_monitor_cck_pd` | `0`, `0x10`-`0xf0` | Fixed CCK packet-detection threshold. Higher = less sensitive. `0` = stock. |

```
P=/sys/module/88XXau_winject/parameters
echo 0x3c > $P/rtw_monitor_igi
echo 0x8a > $P/rtw_monitor_cck_pd
echo 0    > $P/rtw_monitor_igi        # back to stock behaviour
```

The overrides are applied only while the interface is in monitor mode and take
effect on the next DIG write (within about a second). Station mode is untouched.

A higher IGI trades weak-signal sensitivity for fewer false alarms: a link works
down to about (IGI - 110) - 8 dB. See the measurements in the project's loss tests
for the trade-off on AWUS036ACS (RTL8811AU).

## Module name

This build is the module `88XXau_winject` (USB driver name `rtl88xxau_winject`), so it
can be told apart from the `88XXau_wfb` builds of the upstream wfb fork. Both bind the
same USB IDs and cannot be loaded at the same time: remove the old DKMS package and
blacklist `88XXau`, `88XXau_wfb` and the in-kernel `rtw88_8821au` / `rtw88_8812au`
before loading it. Module options move to the new name, for example
`options 88XXau_winject MaxTxBufLen=32 rtw_monitor_pass_crc_err=1`.
