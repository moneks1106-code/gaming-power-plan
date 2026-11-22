# Gaming Power

**Gaming Power** is a minimal, latency-focused Windows power plan for competitive and high-refresh-rate gaming.

It uses the Windows **High performance** personality and adds twenty-nine targeted overrides, almost all of them for AC power only: aggressive CPU response, no core parking, Windows latency hints, processor idle thresholds that favor shallow idle states, reduced USB/disk/NVMe power-state transitions, no sleep on AC power, and the display off after 10 minutes. The processor settings apply to the P-cores of Intel hybrid CPUs as well. The plan does not disable processor idle states.

Community users have reported a more responsive feel and more consistent host-delivery timing in [InputLab](https://github.com/moneks1106-code/InputLab) (formerly AleksMouseTester). These reports are encouraging, but they are not proof of a universal end-to-end input-latency reduction. Results depend on the CPU, motherboard, BIOS, chipset drivers, USB controller, mouse, polling rate, workload, and Windows build.

Questions, results and feedback: join the [InputLab Discord](https://discord.gg/3kEktvD4zm).

## What's new in v1.2.0

v1.2.0 keeps every setting of v1.1.0 unchanged and adds sixteen values from the tested "Idle 100 + NVMe 0" profile:

- **Processor idle thresholds:** idle promote and idle demote threshold at 100 % (Windows default 60 % / 40 %), on AC and on battery. This is meant to keep the cores in shallow idle states. Idle states stay enabled: "Processor idle disable" is now set to off explicitly.
- **NVMe idle timeouts:** primary and secondary NVMe idle timeout at 0 ms (Windows defaults for High performance: 200 ms / 2000 ms), on AC and on battery.
- **Idle resiliency:** "Deep Sleep" disabled.
- **No sleep on AC power:** sleep states (S1–S3) are not allowed, hybrid sleep is off, and the system does not go back to sleep on its own after an unattended wake.
- **Display:** turns off after 10 minutes and is never dimmed. While the PC is locked, Windows does not turn the display off (lock-screen timeout 0).

Updating from v1.1.0 or v1.0.0: see [Updating from an earlier version](#updating-from-an-earlier-version).

## What changed in v1.1.0

- **Intel hybrid CPUs (12th gen and newer):** Windows keeps separate processor settings for the P-cores (efficiency class 1). v1.0.0 set its processor overrides only for the first class, so on these CPUs the P-cores used whatever Windows or Intel's provisioning supplied. v1.1.0 sets them for the P-cores as well. On CPUs without P- and E-cores, Windows ignores these values: AMD Ryzen and older Intel CPUs behave as with v1.0.0.
- **Core parking is now off explicitly** (minimum unparked cores 100 %, for both core classes). v1.0.0 left it to the High performance personality.

Thanks to the community member whose core parking report led to this fix.

## Download

Download [`Gaming-Power.pow`](./Gaming-Power.pow), or use the asset attached to the [latest release](https://github.com/moneks1106-code/gaming-power-plan/releases/latest).

Published SHA-256:

```text
437BD6D9127DFEB51F8AA42E5E82A3CBC1DE08144780D9C29159A3C38BC9967C
```

The same value is available in [`SHA256SUMS.txt`](./SHA256SUMS.txt).

## Compatibility

Gaming Power is intended for desktop testing on:

- AMD AM4
- AMD AM5
- Modern Intel x64 platforms
- Windows 11 x64

The primary reference system is an AMD Ryzen 7 9800X3D on an AM5 platform. AM4 and Intel results are especially welcome. Windows 10 may import the plan, but it is not a primary validation target and may ignore unsupported settings.

This is an aggressive **AC/desktop** profile. It is not recommended as a general-purpose laptop or battery plan. Since v1.2.0 the processor idle thresholds and NVMe idle timeouts also apply on battery.

**Ryzen X3D with two CCDs** (7900X3D, 7950X3D, 9900X3D, 9950X3D): AMD's chipset driver normally parks the CCD without 3D V-Cache while a game runs. With Gaming Power all cores stay unparked; games are still steered to the V-Cache CCD through AMD's preferred-core ranking, but not strictly. If a game runs worse for you, please report it in the Issues. Single-CCD X3D CPUs (7800X3D, 9800X3D) are not affected.

## Installation

1. Download `Gaming-Power.pow`.
2. Open Windows Terminal or Command Prompt as Administrator.
3. Import the plan using the full path to the downloaded file:

   ```powershell
   powercfg /import "C:\full\path\Gaming-Power.pow"
   ```

4. List the installed plans and find the GUID shown next to `Gaming Power`:

   ```powershell
   powercfg /list
   ```

5. Activate that GUID:

   ```powershell
   powercfg /setactive <GAMING_POWER_GUID>
   ```

You can also activate the imported plan from Windows Power Options.

## Updating from an earlier version

An imported plan does not update itself. To update from v1.1.0 or v1.0.0:

1. Import the new `Gaming-Power.pow` as above. `powercfg /list` then shows two plans named `Gaming Power`.
2. Activate the new one with `powercfg /setactive <NEW_GUID>`.
3. Delete the old one, the one without the `*`: `powercfg /delete <OLD_GUID>`.

## Restore or uninstall

Switch back to Balanced before deleting Gaming Power:

```powershell
powercfg /setactive SCHEME_BALANCED
powercfg /list
powercfg /delete <GAMING_POWER_GUID>
```

Only delete the GUID shown next to `Gaming Power` on your own system.

## Technical profile

The `.pow` file is deliberately small and contains only these explicit overrides. Settings marked "AC + battery" also apply on battery power, all others on AC power only. The first five processor settings are set twice, for both core classes: on an Intel hybrid CPU the second copy applies to the P-cores, elsewhere Windows ignores it.

| Setting | Value | Power | Intent / trade-off |
| --- | ---: | --- | --- |
| Minimum processor performance state | 100% | AC | Keeps active CPU performance aggressive; may increase power and temperature |
| Processor energy performance preference (EPP) | 0 | AC | Favors performance over efficiency |
| Performance on latency sensitivity hint | 100% | AC | Requests maximum performance for supported Windows latency hints |
| EPP on latency sensitivity hint | 0 | AC | Favors performance while a supported latency hint is active |
| Core parking minimum cores | 100% | AC | No core parking: every core stays unparked; on dual-CCD X3D see Compatibility |
| Processor idle promote threshold | 100% | AC + battery | Together with the demote threshold, meant to keep the cores in shallow idle states; increases idle power and temperature. Microsoft's description of the two thresholds is ambiguous, and the effect depends on the platform |
| Processor idle demote threshold | 100% | AC + battery | See idle promote threshold |
| Processor idle disable | Off | AC | Explicit: processor idle states stay enabled |
| Idle resiliency "Deep Sleep" | Disabled | AC | Part of the tested profile; Microsoft does not document this setting |
| Disk idle timeout | Never | AC | Avoids disk spin-down or idle timeout transitions |
| Primary and secondary NVMe idle timeout | 0 ms | AC + battery | Idle time before Windows may move an NVMe drive to a low-power idle state; meant to avoid NVMe idle transitions. Microsoft documents the range 0–60000 ms but not how 0 is treated. The NVMe latency tolerance settings, which also limit these transitions, stay at the Windows defaults |
| USB selective suspend | Disabled | AC | Avoids USB idle suspend/resume transitions; increases idle power |
| USB 3 link power management | Off | AC | Avoids USB 3 U1/U2 link transitions; applies only to relevant links |
| Allow sleep states (S1–S3) | Off | AC | The PC does not go to sleep on AC power |
| Hybrid sleep | Off | AC | No hibernation file written on sleep |
| Unattended sleep timeout | 0 (never) | AC | No automatic return to sleep after an unattended wake (wake timer, Wake-on-LAN) |
| Turn off display after | 10 min | AC | Display power saving stays on |
| Dim display after | 0 (never) | AC | Only relevant to displays with brightness control |
| Display off timeout on the lock screen | 0 (never) | AC | The display stays on while the PC is locked; see trade-offs |

The plan does **not** disable processor idle states (it changes only the thresholds for moving between them), install software, change HPET/BCD timer options, modify mouse queue sizes, change USB polling rate, or alter mouse sensitivity.

Because the file uses the High performance personality, settings not explicitly included in the file are supplied by Windows and platform-specific provisioning. Effective values can therefore differ between AMD and Intel systems or between Windows builds. The overrides above are explicit for both core classes since v1.1.0, so they no longer depend on that provisioning.

## What InputLab can and cannot show

[InputLab](https://github.com/moneks1106-code/InputLab) (formerly AleksMouseTester) is useful for examining how evenly mouse input is delivered to the Windows host. It does not directly measure full click-to-photon latency, game-engine latency, display latency, or total system input lag.

A cleaner polling graph is useful evidence about report consistency, but it should not be presented by itself as proof that the complete input path became faster.

For a useful comparison:

1. Keep the same mouse, USB port, polling rate, DPI, game settings, frame cap, and background processes.
2. Compare Balanced, High performance, and Gaming Power.
3. Alternate the test order and repeat each plan several times.
4. Record Windows build, CPU, motherboard, BIOS, chipset driver, mouse, firmware, and polling rate.
5. Share raw results or screenshots instead of only the best run.

Community test results can be submitted through GitHub Issues.

## Important trade-offs

- Higher idle and light-load power consumption is expected.
- CPU temperature and fan activity may increase.
- Microsoft generally recommends leaving USB selective suspend enabled; this plan disables it on AC as an experimental latency/stability trade-off.
- Disabling USB and link power savings is not beneficial on every device.
- Active gaming devices may already remain fully powered, making some USB changes irrelevant.
- Microsoft recommends changing the processor idle thresholds only with guidance from the CPU vendor, because unsuitable values can hurt both efficiency and responsiveness. Compare against v1.1.0 if something feels worse.
- The idle thresholds and NVMe idle timeouts apply on battery as well; on a laptop they cost battery life.
- The PC does not go to sleep while on AC power. Shut it down when you leave it for longer, or switch to Balanced.
- While the PC is locked, Windows does not turn the display off. On OLED monitors, turn the monitor off yourself to avoid burn-in.
- Average FPS may be unchanged even when the system feels more responsive.
- No performance or latency improvement is guaranteed.

Microsoft documents the relevant mechanisms in its [processor power management](https://learn.microsoft.com/windows-hardware/customize/power-settings/configure-processor-power-management-options), [latency hint](https://learn.microsoft.com/windows-hardware/customize/power-settings/options-for-perf-state-engine-latencyhintepp), [idle promote threshold](https://learn.microsoft.com/windows-hardware/customize/power-settings/options-for-perf-state-engine-idlepromotethreshold), [idle demote threshold](https://learn.microsoft.com/windows-hardware/customize/power-settings/options-for-perf-state-engine-idledemotethreshold), [NVMe power management](https://learn.microsoft.com/windows-hardware/design/component-guidelines/power-management-for-storage-hardware-devices-nvme), [sleep states](https://learn.microsoft.com/windows-hardware/customize/power-settings/sleep-settings-allow-sleep-states), [USB selective suspend](https://learn.microsoft.com/windows-hardware/drivers/usbcon/usb-selective-suspend), and [USB 3 link power management](https://learn.microsoft.com/windows-hardware/drivers/usbcon/usb-3-0-lpm-mechanism-) documentation.

## License

Released under the [MIT License](./LICENSE).
