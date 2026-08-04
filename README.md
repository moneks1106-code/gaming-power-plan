# Gaming Power

**Gaming Power** is a minimal, latency-focused Windows power plan for competitive and high-refresh-rate gaming.

It uses the Windows **High performance** personality and adds seven targeted AC-power overrides for aggressive CPU response, Windows latency hints, and reduced USB/storage power-state transitions. The plan does not disable processor idle states.

Community users have reported a more responsive feel and more consistent host-delivery timing in [AleksMouseTester](https://github.com/moneks1106-code/AleksMouseTester). These reports are encouraging, but they are not proof of a universal end-to-end input-latency reduction. Results depend on the CPU, motherboard, BIOS, chipset drivers, USB controller, mouse, polling rate, workload, and Windows build.

## Download

Download [`Gaming-Power.pow`](./Gaming-Power.pow), or use the asset attached to the [latest release](https://github.com/moneks1106-code/gaming-power-plan/releases/latest).

Published SHA-256:

```text
82622217CE9490D737F8F548B009F1E19D7220DC0FB02E3CE72F33C3429D1D44
```

The same value is available in [`SHA256SUMS.txt`](./SHA256SUMS.txt).

## Compatibility

Gaming Power is intended for desktop testing on:

- AMD AM4
- AMD AM5
- Modern Intel x64 platforms
- Windows 11 x64

The primary reference system is an AMD Ryzen 7 9800X3D on an AM5 platform. AM4 and Intel results are especially welcome. Windows 10 may import the plan, but it is not a primary validation target and may ignore unsupported settings.

This is an aggressive **AC/desktop** profile. It is not recommended as a general-purpose laptop or battery plan.

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

## Restore or uninstall

Switch back to Balanced before deleting Gaming Power:

```powershell
powercfg /setactive SCHEME_BALANCED
powercfg /list
powercfg /delete <GAMING_POWER_GUID>
```

Only delete the GUID shown next to `Gaming Power` on your own system.

## Technical profile

The `.pow` file is deliberately small and contains only these seven explicit AC overrides:

| Setting | AC value | Intent / trade-off |
| --- | ---: | --- |
| Minimum processor performance state | 100% | Keeps active CPU performance aggressive; may increase power and temperature |
| Processor energy performance preference (EPP) | 0 | Favors performance over efficiency |
| Performance on latency sensitivity hint | 100% | Requests maximum performance for supported Windows latency hints |
| EPP on latency sensitivity hint | 0 | Favors performance while a supported latency hint is active |
| USB selective suspend | Disabled | Avoids USB idle suspend/resume transitions; increases idle power |
| USB 3 link power management | Off | Avoids USB 3 U1/U2 link transitions; applies only to relevant links |
| Disk idle timeout | Never | Avoids disk spin-down or idle timeout transitions on AC |

The plan does **not** disable processor idle states, install software, change HPET/BCD timer options, modify mouse queue sizes, change USB polling rate, or alter mouse sensitivity.

Because the file uses the High performance personality, settings not explicitly included in the file are supplied by Windows and platform-specific provisioning. Effective values can therefore differ between AMD and Intel systems or between Windows builds.

## What AleksMouseTester can and cannot show

[AleksMouseTester](https://github.com/moneks1106-code/AleksMouseTester) is useful for examining how evenly mouse input is delivered to the Windows host. It does not directly measure full click-to-photon latency, game-engine latency, display latency, or total system input lag.

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
- Average FPS may be unchanged even when the system feels more responsive.
- No performance or latency improvement is guaranteed.

Microsoft documents the relevant mechanisms in its [processor power management](https://learn.microsoft.com/windows-hardware/customize/power-settings/configure-processor-power-management-options), [latency hint](https://learn.microsoft.com/windows-hardware/customize/power-settings/options-for-perf-state-engine-latencyhintepp), [USB selective suspend](https://learn.microsoft.com/windows-hardware/drivers/usbcon/usb-selective-suspend), and [USB 3 link power management](https://learn.microsoft.com/windows-hardware/drivers/usbcon/usb-3-0-lpm-mechanism-) documentation.

## License

Released under the [MIT License](./LICENSE).
