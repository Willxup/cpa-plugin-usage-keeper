# CPA Usage Keeper Plugin

CPA plugin that adds a `Keeper` resource entry to CPAMC and opens
`cpa-usage-keeper` inside the management-center plugin iframe.

The plugin does not proxy Keeper APIs and does not create Keeper sessions. It
only registers a browser resource route and redirects that route to the
configured Keeper application URL with `embed=cpamc`.

## Configuration

Enable the plugin in CPA and configure the Keeper application root URL:

```yaml
plugins:
  enabled: true
  configs:
    keeper:
      enabled: true
      priority: 1
      keeper_url: "https://cpa.example.com/keeper/"
```

`keeper_url` must be a full `http://` or `https://` application root URL. It
must not include query parameters or fragments.

Examples:

```text
https://cpa.example.com/keeper/
https://keeper.example.com/
```

For the most reliable embedded login experience, deploy Keeper under the same
browser origin as CPAMC, preferably as a subpath such as `/keeper/`.

## Management API

The plugin declares the `management_api` capability and registers a single
browser resource route. It does not add JSON management endpoints and does not
proxy requests to Keeper.

## Resource Page

The plugin registers one Management API resource:

```text
GET /v0/resource/plugins/keeper/open
```

CPAMC renders plugin resources in an iframe. This resource page immediately
navigates the current iframe to:

```text
<keeper_url>?embed=cpamc
```

If the plugin is missing `keeper_url`, or the value is invalid, the resource
page renders a small configuration message instead.

## Cookie Boundary

Keeper owns its own authentication. Cross-site iframe login may fail when the
browser blocks third-party cookies or when Keeper is deployed with incompatible
cookie settings. This plugin does not implement a token bridge or popup login
flow.

## Build

```bash
make test
make build
make package
```

On macOS this creates:

```text
dist/keeper.dylib
dist/keeper_0.1.0_darwin_arm64.zip
dist/keeper_0.1.0_darwin_arm64.zip.sha256
```

Install locally by copying the dynamic library to CPA's plugin discovery
directory, for example:

```bash
mkdir -p /path/to/CLIProxyAPI/plugins/darwin/arm64
cp dist/keeper.dylib /path/to/CLIProxyAPI/plugins/darwin/arm64/keeper.dylib
```

Target platform, output directory, and runtime plugin version can be overridden:

```bash
make build GOOS=darwin GOARCH=arm64 BUILD_DIR=/path/to/plugins/darwin/arm64
make package VERSION=0.1.0
```

## Plugin Store Release

For plugin-store installation, each GitHub release must include:

```text
keeper_<version>_<goos>_<goarch>.zip
checksums.txt
```

The release workflow publishes the following platform archives:

Expected asset names for version `0.1.0`:

```text
keeper_0.1.0_darwin_amd64.zip
keeper_0.1.0_darwin_arm64.zip
keeper_0.1.0_freebsd_amd64.zip
keeper_0.1.0_linux_amd64.zip
keeper_0.1.0_linux_arm64.zip
keeper_0.1.0_windows_amd64.zip
keeper_0.1.0_windows_arm64.zip
checksums.txt
```

Each zip must contain the dynamic library at the zip root:

- Darwin: `keeper.dylib`
- FreeBSD: `keeper.so`
- Linux: `keeper.so`
- Windows: `keeper.dll`

`checksums.txt` must be in sha256sum format.

Generate a local aggregate checksum file with:

```bash
make checksums VERSION=0.1.0
```

After publishing the release, update
`router-for-me/CLIProxyAPI-Plugins-Store` with this plugin's registry entry.
