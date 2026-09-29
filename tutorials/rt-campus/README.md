# Running the campus notebook in VS Code

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
