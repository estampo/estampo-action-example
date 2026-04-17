# estampo GitHub Action Example

Slice 3D models on every push and PR using [estampo](https://github.com/estampo/estampo) and OrcaSlicer.

**Use this template** to add reproducible slicing to your own repo.

## What it does

On every push to `main` or pull request, GitHub Actions:

1. Pulls the estampo Docker image (OrcaSlicer + bambox bundled)
2. Loads the STL, arranges parts on the build plate, slices with OrcaSlicer
3. Repacks the output into a Bambu-compatible `.gcode.3mf`
4. Posts a PR comment with print metrics (time, filament, layers)
5. Uploads the sliced output as a build artifact

## Files

| File | Purpose |
|------|---------|
| `estampo.toml` | Pipeline config: parts, slicer settings, pack stage |
| `cube_10mm.stl` | Example model (replace with your own) |
| `.github/workflows/slice.yml` | GitHub Actions workflow |

## Quick start

1. Click **Use this template** (or fork this repo)
2. Replace `cube_10mm.stl` with your model(s)
3. Edit `estampo.toml` to match your printer and settings
4. Push a commit and watch the Actions tab

## The workflow

```yaml
- name: Slice model
  uses: estampo/estampo/action@main
  with:
    config: estampo.toml
    comment: "true"
```

The action pulls the Docker image, runs the full pipeline, extracts metrics, and posts a PR comment. It also exposes outputs (`print-time`, `filament-grams`, `layer-count`, etc.) for downstream steps.

## The config

```toml
[pipeline]
stages = ["load", "arrange", "plate", "slice", "pack"]

[slicer]
engine = "orca"
version = "2.3.1"

[slicer.orca]
printer = "Bambu Lab P1S 0.4 nozzle"
process = "0.20mm Standard @BBL X1C"
filaments = ["Generic PLA @base"]

[[parts]]
file = "cube_10mm.stl"
copies = 2

[pack]
command = "bambox repack {output_dir}/plate_sliced.gcode.3mf"
output = "{output_dir}/plate_sliced.gcode.3mf"
```

Key settings to change for your setup:
- **`printer`** -- your printer profile (run `estampo init` to browse available profiles)
- **`process`** -- print quality preset
- **`filaments`** -- material(s) you're using
- **`file`** -- path to your STL/STEP/3MF model

## Running locally

```bash
pipx install estampo
estampo run estampo.toml
```

Or with Docker directly:

```bash
docker run --rm -v "$PWD:/project" --workdir /project \
  ghcr.io/estampo/estampo:orca-2.3.1 \
  run estampo.toml --local -v
```

## Links

- [estampo](https://github.com/estampo/estampo) -- the build system for reproducible 3D prints
- [Action reference](https://github.com/estampo/estampo/blob/main/action/action.yml) -- all inputs and outputs
- [Config docs](https://github.com/estampo/estampo/blob/main/docs/config.md) -- full TOML reference
