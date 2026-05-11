# RuView Linux Live WiFi Experiment

This repository contains the working patch produced from the local RuView run that switched the sensing server from simulated data to live Linux WiFi RSSI data.

Source baseline:

- Upstream working tree: `https://github.com/Adaptare-Vero/RuView.git`
- Local commit created: `9f92275998d346cfb57801a69c49606e4adeb79b`
- Patch file: `patches/ruview-linux-live-wifi.patch`

What the patch does:

- Adds a Linux RSSI collector for `/proc/net/wireless` in `wifi-densepose-sensing-server`.
- Resolves the connected SSID with `iw dev <iface> link`.
- Makes `--source wifi` and auto-detect use live Linux WiFi on Linux hosts.
- Labels live data as `linux-wifi:<ssid>`.
- Updates the browser UI source mapper so `linux-wifi:<ssid>` is treated as live data rather than simulated data.

Validated locally:

- Interface: `wlp3s0`
- SSID: `kitchen side`
- Backend: `http://localhost:3000`
- UI: `http://localhost:3000/ui/index.html`
- Health source: `linux-wifi:kitchen side`
- Live RSSI observed around `-71` to `-73 dBm`

Apply and run:

```bash
git clone https://github.com/Adaptare-Vero/RuView.git
cd RuView
git am /path/to/ruview-linux-live-wifi.patch
. "$HOME/.cargo/env"
cd v2
cargo build -p wifi-densepose-sensing-server --no-default-features
RUST_LOG=info ./target/debug/sensing-server \
  --source wifi \
  --tick-ms 500 \
  --ui-path ../ui \
  --http-port 3000 \
  --ws-port 3001 \
  --udp-port 5005
```

Then open:

```text
http://localhost:3000/ui/index.html
```

Note: this uses live RSSI from the connected Linux WiFi interface, not full CSI. Full DensePose-grade through-wall pose sensing still requires CSI-capable hardware such as ESP32-S3 nodes.
