# drawio-skill — From Text to Professional Diagrams

[中文文档](README_CN.md) | [Online Docs](https://agents365-ai.github.io/drawio-skill/)

## What it does

- Generates `.drawio` XML files from natural language descriptions
- Exports diagrams to PNG, SVG, PDF, or JPG using the native draw.io desktop CLI
- **6 diagram type presets**: ERD, UML Class, Sequence, Architecture, ML/Deep Learning, Flowchart — with preset shapes, styles, and layout conventions
- **Animated connectors** (`flowAnimation=1`) for data-flow and pipeline diagrams (visible in SVG and draw.io desktop)
- **ML model diagram support** with tensor shape annotations `(B, C, H, W)` — ideal for NeurIPS/ICML/ICLR papers
- **Grid-aligned layout** — all coordinates snap to 10px multiples for clean alignment
- **Browser fallback** — generates diagrams.net URLs when the desktop CLI is unavailable
- Iterative design: preview, get feedback, and refine diagrams until they look right
- **Auto-launch** draw.io desktop after export for manual fine-tuning
- Triggers automatically when diagrams would help explain complex systems
- **Style presets (v1.3 new)** — teach the skill your visual style from a `.drawio` file or image, save it by name, and apply it to future diagrams. See `## Style Presets` in SKILL.md.
- **Custom output directory (v1.4 new)** — ask for any output path (e.g. `./artifacts/`, `docs/images/`) and the skill will `mkdir -p` and export there; ideal for CI/CD artifact pipelines.

## Comparison

See [COMPARISON.md](COMPARISON.md) for side-by-side tables vs. native agents and vs. other draw.io skills/tools (jgraph/drawio-mcp, bahayonghang/drawio-skills, GBSOSS/ai-drawio), plus the key-advantages summary.

## Supported diagram types

- **Architecture**: microservices, cloud (AWS/GCP/Azure), network topology, deployment — with tier-based swimlanes and hub-center strategy
- **ML / Deep Learning**: Transformer, CNN, LSTM, GRU architectures — with tensor shape annotations and layer-type color coding
- **Flowcharts**: business processes, workflows, decision trees, state machines — with semantic shape types (parallelogram I/O, diamond decisions)
- **UML**: class diagrams (inheritance/composition/aggregation arrows), sequence diagrams (lifelines, activation boxes)
- **Data**: ER diagrams (table containers, PK/FK notation), data flow diagrams (DFD)
- **Other**: org charts, mind maps, wireframes

## How it works

<p align="center">
  <img src="assets/workflow.png" width="420" alt="Workflow">
</p>

## Installation

Two steps — install the draw.io CLI first, then drop the skill into your host:

1. **[Install draw.io desktop](INSTALL_CLI.md)** — per-platform recipes for macOS / Windows / Linux.
2. **[Install the skill](INSTALL_SKILL.md)** — plugin marketplace (recommended), manual clone, and update commands.

## Usage

See [USAGE.md](USAGE.md) for natural-language prompts, a microservices walkthrough, and topology demos (star / layered / ring).

## Style Presets

Style presets let you capture and reuse a visual style across diagrams. When a preset is active, it replaces the built-in color palette, shape vocabulary, fonts, and edge defaults.

### Built-in presets

| Name | Description |
|------|-------------|
| `default` | Clean blue/green/yellow palette matching the built-in conventions |
| `corporate` | Muted, professional palette suited for business presentations |
| `handdrawn` | Sketch-style strokes for informal or whiteboard-style diagrams |

### Apply a preset to a diagram

```
Draw a microservices architecture using my "corporate" style
```

Or set a default so all future diagrams use it automatically:

```
Make "corporate" my default style
```

### Learn your style from a file

Point the skill at any `.drawio` file or flat image:

```
Learn my style from ~/diagrams/brand.drawio as "mybrand"
Learn my style from ~/diagrams/screenshot.png as "mybrand"
```

The skill extracts colors, shapes, fonts, and edge style, renders a sample diagram for preview, and saves to `~/.drawio-skill/styles/mybrand.json` only after you approve.

### Manage presets

| What you say | What happens |
|---|---|
| "list my styles" | Shows all user and built-in presets in a table |
| "show my `<name>` style" | Pretty-prints the preset JSON |
| "make `<name>` the default" | Sets it as the active default for all diagrams |
| "remove default" | Clears the default (reverts to built-in conventions) |
| "delete `<name>`" | Deletes the user preset (prompts for confirmation) |
| "rename `<a>` to `<b>`" | Renames a user preset |

## License

MIT

## Support

If this skill helps you, consider supporting the author:

<table>
  <tr>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Agents365-ai/images_payment/main/qrcode/wechat-pay.png" width="180" alt="WeChat Pay">
      <br>
      <b>WeChat Pay</b>
    </td>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Agents365-ai/images_payment/main/qrcode/alipay.png" width="180" alt="Alipay">
      <br>
      <b>Alipay</b>
    </td>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Agents365-ai/images_payment/main/qrcode/buymeacoffee.png" width="180" alt="Buy Me a Coffee">
      <br>
      <b>Buy Me a Coffee</b>
    </td>
    <td align="center">
      <img src="https://raw.githubusercontent.com/Agents365-ai/images_payment/main/awarding/award.gif" width="180" alt="Give a Reward">
      <br>
      <b>Give a Reward</b>
    </td>
  </tr>
</table>

## Author

**Agents365-ai**

- Bilibili: https://space.bilibili.com/441831884
- GitHub: https://github.com/Agents365-ai
