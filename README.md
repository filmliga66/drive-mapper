# DriveMapper

A small Windows command-line utility that maps local drive letters to network shares (or local paths) using `subst.exe` and assigns them friendly labels via the registry. Built primarily so Adobe Lightroom can find catalog folders behind a stable drive letter even when the underlying network path changes.

## Why?

Lightroom binds catalogs and previews to drive letters. A regular Windows mapped network drive (`net use`) is not always treated identically to a local drive by every application. `subst` produces what looks like a local drive — which Lightroom is happy with — and DriveMapper automates creating those substitutions on every login, optionally waiting for the file server to be reachable first.

## How it works

For every entry in `DriveMappings.xml` the tool:

1. If the path is a UNC path (`\\server\share`), pings the host until it replies (up to ~4 minutes) so it works even when started before the network is fully up.
2. Runs `subst <Drive> <NetworkPath>` to create the drive letter.
3. Writes the configured label into `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2\...\_LabelFromReg` so Explorer shows a friendly name.

## Installation

### Option 1 — Download a release (recommended)

1. Grab the latest `FL.DriveMapper-<version>-win-x64.zip` from the [Releases page](../../releases) (use `win-arm64` on ARM machines).
2. Extract it somewhere stable, e.g. `C:\Tools\DriveMapper\`.
3. Edit `DriveMappings.xml` to match your shares (see below).
4. Register it to run at logon — see *Run at logon* below.

The published binary is self-contained: no .NET runtime install required.

### Option 2 — Build from source

Requirements: .NET 10 SDK.

```powershell
dotnet publish FL.DriveMapper/FL.DriveMapper.csproj -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true -o publish
```

The resulting `publish/FL.DriveMapper.exe` is a single self-contained executable.

## Configuration

`DriveMappings.xml` lives next to the executable. Example:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<DriveMappings>
  <Mappings>
    <DriveMapInfo>
      <Drive>L:</Drive>
      <NetworkPath>\\sepp\projekte</NetworkPath>
      <Label>Projekte Lightroom</Label>
    </DriveMapInfo>
    <DriveMapInfo>
      <Drive>M:</Drive>
      <NetworkPath>\\sepp\medien</NetworkPath>
      <Label>Medien</Label>
    </DriveMapInfo>
  </Mappings>
</DriveMappings>
```

| Field         | Meaning                                                  |
| ------------- | -------------------------------------------------------- |
| `Drive`       | The drive letter to create, e.g. `L:`                    |
| `NetworkPath` | The target — either a UNC path or a local folder         |
| `Label`       | The display name shown in Explorer for the mapped drive  |

## Run at logon

A ready-to-import Windows Task Scheduler definition is shipped as `TaskSchedulerSetting.xml`.

1. Open **Task Scheduler** → **Import Task...**
2. Select `TaskSchedulerSetting.xml`.
3. Adjust the **Action → Program/script** path to point at your `FL.DriveMapper.exe`.
4. On the **General** tab, change **User** to your own account.

The task triggers at logon and runs with least-privilege; no admin rights are needed since `subst` and the relevant registry keys are per-user.

## Logs

`log4net` is used for diagnostics. Drop a `log4net.config` file next to the executable and configure appenders as you like; without it the tool runs silently.

## Releases & versioning

Pushing a tag `vX.Y.Z` to `master` triggers the `Release` workflow which builds self-contained single-file executables for `win-x64`, `win-x86`, and `win-arm64` and publishes a GitHub Release with one zip per architecture.

## Contributing

Dependencies are kept current automatically via Dependabot. Minor and patch NuGet updates as well as GitHub Actions updates are auto-merged once CI passes; major upgrades are opened as PRs for review.

## License

MIT — see [LICENSE](LICENSE).
