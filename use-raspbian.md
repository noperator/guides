List displays.
```
tvservice -l
2 attached device(s), display ID's are : 
Display Number 2, type HDMI 0
Display Number 7, type HDMI 1

tvservice -l | grep '^Display' | while read ENTRY; do
    echo "$ENTRY" | tr '\n' ' '
    tvservice -s -v $(echo "$ENTRY" | awk '{print $3}' | tr -d ',')
done

Display Number 2, type HDMI 0 state 0x2 [TV is off]
Display Number 7, type HDMI 1 state 0x6 [DVI DMT (82) RGB full 16:9], 1920x1080 @ 60.00Hz, progressive
```

Power specific display off/on. [More info](https://raspberrypi.stackexchange.com/a/29149).
```
tvservice -v 7 -o
Powering off HDMI

tvservice -v 7 -p
Powering on HDMI with preferred settings

fbset -depth 8 && fbset -depth 16  # cycle color depth
```

Send both displays into low power mode. See [differences](https://github.com/raspberrypi/userland/issues/447#issuecomment-379460275) between `vcgencmd` and `tvservice`.
```
vcgencmd display_power 0
vcgencmd display_power 1
```
