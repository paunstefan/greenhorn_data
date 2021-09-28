# Decoding SSTV on Linux

## Software needed
* Pulseaudio (+ pavucontrol)
* QSSTV

Firstly, load the needed Pulseaudio module (modify `default.pa` if you need it permanently)

```bash
pactl load-module module-null-sink sink_name=virtual-cable
```

Run `pavucontrol` and under the Recording tab switch the QSSTV to capture from *Monitor of Null Output*.

Now go to the directory with your audio file run it through the virtual audio cable:

```bash
paplay -d virtual-cable [filename]
```

## Useful websites

* [Decoding SSTV from a file](https://www.chonky.net/hamradio/decoding-sstv-from-a-file-on-a-linux-system)
* [Ham software for Linux](http://users.telenet.be/on4qz/)
* [WebSDR](http://websdr.org/)