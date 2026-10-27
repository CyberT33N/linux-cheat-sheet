# PulseAudio Configuration & Recovery

## Route playback through PulseEffects

- Open the PulseEffects app
  - Settings -> Audio -> Put every playback stream to PulseEffects. Make sure that at the top, under Playback devices, the PulseEffects app is at 100% volume and you selected your soundcard
    - The PulseEffects playback stream goes to your soundcard
      - Open PulseAudio Volume Control and go to the Playback tab
        - The Pulse Effects playback stream goes to your soundcard; everything else goes to PulseEffects (app)
        - From now on always use PulseAudio Volume Control instead of the regular audio settings

## Sound not working anymore after sign-out

### Option 1

- Turn the default audio adapter off, then on again, and start PulseEffects again

### Option 2

- Open PulseAudio Volume Control (search it via the menu bar), then change the audio output for the application. If this does not work, try `killall pulseaudio`, sign out again, and try changing the audio output once more

## Restart pulseaudio

```shell
pulseaudio -k && pulseaudio --start
```