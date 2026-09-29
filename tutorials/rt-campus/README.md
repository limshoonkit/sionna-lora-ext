# Running the campus notebooks in VS Code

## Six-node 923 MHz peer-to-peer study

Open [1_peer_to_peer_lora_923mhz.ipynb](1_peer_to_peer_lora_923mhz.ipynb), select
`nvidia/.venv-lora/bin/python`, and **Run All**. It is independent of notebook 0.
The default uses CPU ray tracing and static plots, so it works even if the
interactive 3D widget does not render. Allow time for the link and map cells.

- Six peers, three per floor, seeded placement with 5 m minimum same-floor spacing.
- Allowed subsets of corridors, library, atrium/walkway and public computing suite;
  floor-support and clearance checks reject voids and occupied geometry.
- Thirty directed PHY links, RSS and SF7/SF12 margin matrices, distance scatter,
  a static 3D node view, and floor-slice signal maps from virtual receivers.
- Edit `N_NODES` and region `count` entries together to change node count.
  Change `MAP_TX_INDEX` for another source and `MAP_STEP_M` for a finer map.
  Rerun from the configuration cell after changing placement or radio settings.
- `OPEN_INTERACTIVE_PREVIEW = True` enables the optional **in-notebook** viewport;
  it does not open a separate window. Static results do not depend on it.

The notebook includes placement and free-space power self-checks. Results are
predictions with explicit hardware assumptions, not measured RSSI, packet delivery,
or channel congestion. Hardware calibration and solver convergence still matter.
The initial run finds large forward/reverse differences on some links; the
notebook flags these as unconverged rather than claiming reliable coverage.

## Original campus/AP notebook

1. Open the `nvidia` workspace folder. Campus assets are in `sionna-lora-ext/assets`.
2. Open `0_indoor_campus_radio_environment.ipynb` and select the **.venv-lora**
   kernel (`nvidia/.venv-lora/bin/python`).
3. Run the cells in order with `OPEN_INTERACTIVE_PREVIEW = True`. The interactive
   view appears after the AP candidates in section 5; the earlier reference image
   is static. `RENDER_STATIC` and `RUN_RADIO_MAP` are optional and default to `False`.

## Preview troubleshooting

- **Widget loading errors:** the workspace `.vscode/settings.json` sets
  `"jupyter.widgetScriptSources": []` to use widget scripts installed in the kernel
  environment, avoiding CDN downloads that timed out on this machine. After
  changing this setting, run **Developer: Reload Window** and rerun the cells.
- **Controls but no 3D canvas:** check `code --status`. If WebGL is disabled because
  OpenGL context creation failed, save your work, fully quit **all** VS Code
  windows, then launch from an external terminal:

  ```bash
  code --use-gl=angle --use-angle=swiftshader /path/to/nvidia
  ```

  Rerun the notebook. **Reload Window alone does not apply startup flags.** This
  enables CPU-based 3D rendering, which may be slower. Run `code --status` after
  relaunching: `Process Argv` must contain both flags and `webgl` must be `enabled`.
  If the flags are absent, the command reused an existing VS Code instance or
  another launcher omitted them. Save all work, use **File > Exit**, and run the
  command above from an external terminal; use `/snap/bin/code` in place of `code`
  to bypass shell aliases on this machine.

On this machine, the flags are already saved in the Bash `code` alias and user
desktop launchers. New terminals and normal app launches include them; existing
Bash terminals can run `source ~/.bash_aliases`. This VS Code build does not
accept these two flags in `argv.json`.

Reference: [Chromium SwiftShader flags](https://github.com/chromium/chromium/blob/main/docs/gpu/swiftshader.md).

## Selected hardware for the peer-to-peer LoRa experiment

Use identical RAK4631(H) / SX1262 nodes with the 902-930 MHz RAKARJ10 external
antenna connected through a compatible pigtail. The selected antenna is the
2 dBi RP-SMA product, not the RAKARB03 PCB antenna also documented in `ref/`.
This baseline is implemented in `1_peer_to_peer_lora_923mhz.ipynb`. Notebook 0
retains its separate AP-candidate experiment.

| Parameter | Baseline | Basis |
| --- | --- | --- |
| Carrier | 923 MHz | Prepared campus scene |
| Bandwidth | 125 kHz | Initial experiment choice |
| Conducted transmit power | +14 dBm | Initial experiment choice, not the module maximum |
| Spreading factors | SF7 and SF12 initially | Explicit sensitivity anchors in Semtech Table 3-8 |
| Coding rate | 4/5 | Semtech sensitivity test conditions |
| Payload / preamble | 64 bytes / 8 symbols | Semtech sensitivity test conditions |
| Header / payload CRC | Explicit / enabled | Semtech sensitivity test conditions |
| Receiver mode | Boosted gain | Required for the sensitivity anchors below |
| Typical chip sensitivity | SF7: -124 dBm; SF12: -137 dBm | 125 kHz, 1% PER, split RF paths, RF-switch insertion loss excluded |
| Antenna peak gain at 923 MHz | Approximately 1.96 dBi | Linear interpolation between 2.00 dBi at 922 MHz and 1.92 dBi at 924 MHz; not a measured 923 MHz value |
| Antenna orientation | Upright, aligned linear polarization initially | Controlled placement assumption |
| Pigtail/connector loss | Provisional 1 dB per endpoint; compare 0/1/2 dB | Assumption until cable assembly loss is known |

The Semtech sensitivity values are typical chip specifications under the test
conditions on page 16 and Table 3-8 on page 20, including 25 degrees C and listed
RF test frequencies including 915 MHz. Their use at 923 MHz is a nearby-band
baseline, not a measured RAK4631 sensitivity or a complete PER curve. Account
for module receive-path losses when comparing antenna-port power with these
chip-level thresholds. Do not invent datasheet values for intermediate SFs.

Use the antenna's directional gain in both transmit and receive paths. Peak gain
is not constant over all directions. A vertically polarized dipole-like pattern
scaled to the stated peak is only an initial surrogate; replace it with the
published/measured pattern for installation-specific results. Keep external
pigtail loss separate on each end and do not add antenna efficiency as another
loss when the pattern already represents gain.

Sources:

- [RAK4631 module datasheet](https://docs.rakwireless.com/product-categories/wisblock/rak4631/datasheet/): SX1262, point-to-point support, high-band variant, up to +22 dBm hardware output.
- [Selected antenna product](https://store.rakwireless.com/products/lora-antenna) and [RAKARJ10 antenna datasheet](https://docs.rakwireless.com/product-categories/accessories/rakarj10/datasheet/): connector, polarization, frequency-dependent peak gain and radiation plots.
- [Semtech SX1261/SX1262 Rev. 2.2, December 2024](../../../ref/semtech-sx1261-sx1262-datasheet-2024.pdf): pages 16 and 20 for sensitivity conditions and values.
