# MagicMirror display watchdog

The watchdog follows new kernel journal entries for the Intel i915 error
`CPU pipe A FIFO underrun`. When GNOME reports that the display is on,
it puts the output in standby, waits three seconds, and wakes it. If
another underrun occurs during the cycle or within five seconds after
it finishes, the watchdog checks that the display is on and repeats
the cycle once. The ten-minute cooldown starts with the first cycle
and applies to the whole recovery sequence.

## Prerequisites

Run these commands as `brad` on `magicmirror.local`. The watchdog uses
Python 3, the systemd user manager, access to the kernel journal, and a
GNOME Wayland session with Mutter's display control interface.

The existing helpers live at `~/MagicMirror/monitorOff.sh` and
`~/MagicMirror/monitorOn.sh`. They set Mutter's `PowerSaveMode` to
standby (`1`) and on (`0`), respectively. Keep those paths available
when restoring the watchdog.

## Install

Run this from a shell on the mirror. The two `install` commands copy
the source and unit into the paths used by the service. They replace
existing copies at those paths.

```sh
cd ~/magicmirror-display-watchdog
install -d ~/.local/bin ~/.config/systemd/user
install -m 0644 magicmirror-display-watchdog ~/.local/bin/magicmirror-display-watchdog
install -m 0644 magicmirror-display-watchdog.service ~/.config/systemd/user/magicmirror-display-watchdog.service
systemctl --user daemon-reload
systemctl --user enable --now magicmirror-display-watchdog.service
```

When installing an updated source file while the watchdog is already
running, restart the watchdog to load the new copy.

```sh
systemctl --user restart magicmirror-display-watchdog.service
```

## Check status and logs

```sh
systemctl --user status magicmirror-display-watchdog.service --no-pager
journalctl --user -u magicmirror-display-watchdog.service -n 30 --no-pager
```

A successful recovery log confirms that GNOME reports the output back
on. It does not verify what the physical screen is showing.

## Disable

```sh
systemctl --user disable --now magicmirror-display-watchdog.service
```

If the display is in standby after stopping the watchdog during a
recovery cycle, wake it with `bash ~/MagicMirror/monitorOn.sh`.
