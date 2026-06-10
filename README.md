# luci-app-openclash-build

Build `luci-app-openclash` IPK from source for **aarch64 (armsr/armv8)** OpenWrt/ImmortalWrt devices, using the official SDK.

## Why

The upstream `vernesong/OpenClash` repo only contains the LuCI frontend sources — it ships no prebuilt IPKs. Devices running ImmortalWrt 24.10 SNAPSHOT on `armsr/armv8` (e.g. soft routers, NUCs, ARM mini-PCs) cannot `opkg install` it from the official feeds. The fix is a local SDK build.

`luci-app-openclash` is `PKGARCH:=all` (pure Lua + shell scripts), so a single build works for **any** architecture — including aarch64, x86_64, etc. The target SDK just needs to match the device's `DISTRIB_TARGET`.

## Usage

1. Trigger the workflow from the Actions tab → **Build luci-app-openclash ipk** → Run workflow.
2. Pick a target SDK matching your device:
   - `immortalwrt-2410` — ImmortalWrt 24.10 SNAPSHOT (tested)
   - `immortalwrt-2305` — ImmortalWrt 23.05
   - `openwrt-2410` — upstream OpenWrt 24.10 SNAPSHOT
3. Download the artifact `luci-app-openclash-ipk-<target>` from the run summary.
4. Push to the device:
   ```bash
   scp luci-app-openclash_*.ipk root@10.10.10.130:/tmp/
   ssh root@10.10.10.130 "opkg install /tmp/luci-app-openclash_*.ipk --force-depends"
   ```

## Layout

- `luci-app-openclash/` — upstream sources (as a git submodule or vendored copy).
- `.github/workflows/build.yml` — the SDK build pipeline.

## Notes

- This builds the **LuCI app only**. You still need a clash/mihomo core binary on the device — the LuCI app downloads it on first run.
- For dependencies the package expects (e.g. `dnsmasq-full`, `kmod-tun`, `ruby`, `ruby-yaml`, `bash`, `curl`, `ca-bundle`, `ip-full`, `unzip`), `opkg` will resolve them from your feeds automatically.
- The local SDK probe step in the workflow prints which mirrors are reachable; if all fail, check the Actions log for diagnostics.
