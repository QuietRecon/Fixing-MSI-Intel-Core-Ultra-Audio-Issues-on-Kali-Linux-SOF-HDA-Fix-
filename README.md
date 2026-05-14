# Fixing MSI / Intel Core Ultra Audio Issues on Kali Linux (SOF → HDA Fix)

## Symptoms

* No sound from laptop speakers
* OBS recordings had no microphone audio
* HDMI audio devices only appeared
* Internal speakers/microphone missing
* PipeWire/PulseAudio defaulted to HDMI
* `speaker-test` produced no sound

---

# Diagnostic Commands Used

## Check recording devices

```bash
arecord -l
```

## Check playback devices

```bash
aplay -l
```

## List available ALSA devices

```bash
arecord -L
```

## Check PipeWire / PulseAudio status

```bash
pactl info
```

## List audio sources

```bash
pactl list short sources
```

## Test speaker output

```bash
speaker-test -c 2
```

## Open ALSA mixer

```bash
alsamixer
```

## Open PulseAudio/PipeWire control panel

```bash
pavucontrol
```

## Check kernel audio logs

```bash
sudo dmesg | grep -iE "sof|snd|hda|audio"
```

---

# Root Cause

Kali Linux loaded the Intel SOF audio driver stack (`sof-hda-dsp`) instead of the legacy Intel HDA driver.

This caused:

* HDMI-only audio routing
* missing analog speaker profiles
* microphone routing issues

---

# Final Fix (Worked)

Force Linux to use legacy HDA audio instead of SOF DSP.

## Create override

```bash
echo "options snd-intel-dspcfg dsp_driver=1" | sudo tee /etc/modprobe.d/alsa-base.conf
```

## Rebuild initramfs

```bash
sudo update-initramfs -u
```

## Reboot

```bash
sudo reboot
```

---

# Result After Reboot

Audio devices correctly appeared as:

```text
HDA Intel PCH
ALC256 Analog
analog-stereo
```

Internal speakers and microphone paths returned.

---

# Verify After Reboot

```bash
pactl info
```

```bash
aplay -l
```

---

# Rollback (If Needed)

Remove the override:

```bash
sudo rm /etc/modprobe.d/alsa-base.conf
sudo update-initramfs -u
sudo reboot
```

---

# Notes

* System: MSI Modern 15 H AI C1MG
* CPU: Intel Core Ultra 5 125H
* OS: Kali Linux
* Audio Codec: Realtek ALC256
* Audio Stack: PipeWire + PulseAudio compatibility layer
