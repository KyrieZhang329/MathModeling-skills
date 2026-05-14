---
name: math-figure-generator
description: Generate publication-quality mathematical modeling figures with matplotlib, covering evaluation charts, prediction plots, optimization diagrams, mechanism schematics, heatmaps, and multi-panel layouts. Use when creating or revising figures for contest papers.
license: MIT
---

# Math Figure Generator

A guide for producing publication-quality figures for mathematical modeling contest papers. Every figure starts from a claim, a panel purpose, and an output contract before writing code or styling.

Adapted from the [nature-figure](https://github.com/Yuan1z0825/nature-skills) skill's philosophy: the chart serves the scientific logic first; aesthetic polish is subordinate to making the conclusion clear, defensible, and reviewable.

## First move: figure contract before plotting

Before any code, define the contract:

1. **Core claim**: what one-sentence conclusion this figure defends.
2. **Figure type** from the math modeling figure taxonomy (see below).
3. **Panel map**: what each panel (a, b, c, ...) shows and what unique evidence it carries.
4. **Data source**: exact path to the CSV/mat/json that provides the numbers.
5. **Output contract**: dimensions (figsize), format (SVG primary, PNG secondary), dpi.
6. **Paper placement**: which paper section and whether it's Type 3 (论文图) or Type 4 (附录图).

Delete any panel that does not carry unique evidence. Every panel must answer a different question.

## Math modeling figure taxonomy

| Figure Type | Use When | Common Chart | Example |
|-------------|----------|-------------|---------|
| `evaluation-ranking` | Ranking/scoring alternatives | Horizontal bar, sorted bar, dumbbell | "City A ranks 1st with score 0.88" |
| `evaluation-comparison` | Comparing multiple evaluation methods | Grouped bar, radar, heatmap | "Entropy-TOPSIS differentiates better than equal-weight" |
| `prediction-fit` | Showing prediction vs actual | Scatter + identity line, line + ribbon | "ARIMA captures the trend with RMSE 2.3" |
| `prediction-forecast` | Future projections with uncertainty | Line + confidence band | "Demand projected to reach 120 ± 15 by Q4" |
| `optimization-result` | Allocation, routing, scheduling outcome | Bar, stacked bar, map, network | "Resource allocation: Region A receives 45%" |
| `optimization-sensitivity` | Sensitivity to constraints or parameters | Line plot, tornado, heatmap | "Optimal cost changes <5% under ±10% capacity change" |
| `mechanism-schematic` | Model structure, flow, or process | Flowchart, box diagram, network | "SEIR model with vaccination compartment" |
| `robustness-sensitivity` | Parameter perturbation effects | Line plot, tornado, heatmap | "Top-3 ranking stable under ±10% weight change" |
| `data-exploration` | Data distribution, correlation | Histogram, box, heatmap, pairplot | "Indicators show moderate correlation (ρ < 0.6)" |
| `baseline-comparison` | Baseline vs main model | Side-by-side bar, difference plot | "Main model improves RMSE by 35% over baseline" |
| `workflow-diagram` | Overall solution pipeline | Flowchart, block diagram | "Data → Cleaning → Modeling → Validation → Paper" |

## Python quick-start

```python
import matplotlib as mpl
import matplotlib.pyplot as plt
import numpy as np
from pathlib import Path

# ── MANDATORY: editable SVG text ───────────────────────────────────────
mpl.rcParams.update({
    "font.family": "sans-serif",
    "font.sans-serif": ["Arial", "Helvetica", "DejaVu Sans", "sans-serif"],
    "svg.fonttype": "none",        # keeps text as <text> nodes, not paths
    "pdf.fonttype": 42,            # editable TrueType text in PDF
    "font.size": 10,
    "axes.spines.right": False,
    "axes.spines.top": False,
    "axes.linewidth": 0.8,
    "legend.frameon": False,
})

def save_figure(fig, filename, formats=None, dpi=300):
    """Save figure to specified formats. SVG is always primary."""
    if formats is None:
        formats = ['svg', 'png']
    out = Path(filename)
    out.parent.mkdir(parents=True, exist_ok=True)
    base = out.with_suffix('')
    saved = []
    for fmt in formats:
        if fmt == 'svg':
            fig.savefig(f"{base}.svg", bbox_inches='tight')
        elif fmt == 'pdf':
            fig.savefig(f"{base}.pdf", bbox_inches='tight')
        elif fmt == 'png':
            fig.savefig(f"{base}.png", dpi=dpi, bbox_inches='tight')
        elif fmt == 'tiff':
            fig.savefig(f"{base}.tiff", dpi=dpi, bbox_inches='tight')
        saved.append(f"{base}.{fmt}")
    plt.close(fig)
    return saved
```

## Color palettes

### Semantic palette (default)
```python
PALETTE = {
    'primary':     '#1A6FC4',   # hero method / main series
    'primary_light': '#5B9BD5',
    'baseline':    '#767676',   # baseline / reference
    'baseline_dark': '#4D4D4D',
    'positive':    '#2E9E44',   # improvement / gain
    'positive_light': '#8BCF8B',
    'negative':    '#E53935',   # degradation / loss
    'negative_light': '#F6CFCB',
    'accent1':     '#E28E2C',   # orange
    'accent2':     '#7B5FD6',   # purple
    'accent3':     '#33B5A5',   # teal
    'neutral_light': '#D8D8D8',
    'neutral_mid':   '#A8A8A8',
    'neutral_dark':  '#606060',
}

DEFAULT_COLORS = [
    PALETTE['primary'],
    PALETTE['accent1'],
    PALETTE['accent2'],
    PALETTE['positive'],
    PALETTE['accent3'],
    PALETTE['baseline'],
]
```

### Unified comparison palette (for multi-method comparisons)
```python
PALETTE_COMPARISON = {
    'method1_dark': '#1A6FC4',
    'method1_mid':  '#5B9BD5',
    'method1_light': '#B4D4F0',
    'method2_dark': '#E28E2C',
    'method2_mid':  '#F0B860',
    'method2_light': '#F5D9A0',
    'baseline':      '#767676',
    'delta_up':      '#2E9E44',
    'delta_down':    '#E53935',
    'neutral_bg':    '#F5F5F5',
}
```

**Color rules**:
- `primary` = your main/final model in every panel. Do not recolor the same method differently across panels.
- `baseline` = grey. Baseline is always grey.
- `positive` / `negative` = only for signed deltas (improvement / degradation). Not for general category coloring.
- Related method variants (e.g., baseline, method A, method A+) use the same hue family at different saturation levels.
- Use the unified comparison palette when comparing 2-3 methods across multiple panels.

## Font size hierarchy

| Context | `font.size` |
|---------|-------------|
| Compact paper figure (journal column width) | 7–9 |
| Full-width paper figure | 9–12 |
| Slide/presentation figure | 14–18 |
| Panel labels (a, b, c) | 12–14 |
| Axis labels | same as base or +1 |
| In-bar annotations | 6–8 |
| Legend text | 7–9 |

## Figure sizes (inches)

| Type | figsize (w × h) |
|------|-----------------|
| Single bar chart (horizontal, 5-10 items) | (6, 4) |
| Single bar chart (horizontal, 15+ items) | (6, 7) |
| Grouped bar (3-5 groups, 4-8 categories) | (10, 5) |
| Line trend (single) | (7, 4) |
| Line trend with ribbon | (7, 4.5) |
| Heatmap (square-ish, up to 15×15) | (8, 7) |
| Radar/polar (single) | (6, 6) |
| Scatter (single) | (6, 5) |
| Multi-panel (2×2 grid) | (10, 8) |
| Multi-panel (1×3 row) | (14, 5) |
| Full-page figure (dense, 4-6 panels) | (14, 10) |
| Workflow diagram | (12, 6) |

## Panel labels

```python
def add_panel_label(ax, label, x=-0.06, y=1.02, fontsize=12, fontweight='bold'):
    """Place a bold lowercase panel label (a, b, c) at the top-left of the axes."""
    ax.text(x, y, label, transform=ax.transAxes,
            fontsize=fontsize, fontweight=fontweight,
            va='bottom', ha='left')
```

Use lowercase bold letters (`a`, `b`, `c`). Place consistently at top-left of each axes via `transAxes`.

## Spines and axes

```python
# Always:
ax.spines['right'].set_visible(False)
ax.spines['top'].set_visible(False)

# For emphasis (hero panel):
ax.spines['bottom'].set_linewidth(1.5)
ax.spines['left'].set_linewidth(1.5)

# Tighten y-limits to data range — do not leave empty space from 0
# unless 0 is the natural baseline for the metric.
ax.set_ylim(data_min * 0.95, data_max * 1.05)
```

No gridlines by default. If gridlines are needed (e.g., for readability with many categories), use `ax.grid(axis='y', alpha=0.3, linestyle='--', linewidth=0.5)`.

## Chart-type patterns

### Ranking bar chart (evaluation)
```python
def plot_ranking_bar(ax, labels, scores, colors=None, highlight_top=3):
    """Horizontal bar chart for ranking visualization. Best for evaluation results."""
    n = len(labels)
    y_pos = range(n)
    colors = colors or [PALETTE['primary'] if i < highlight_top else PALETTE['baseline']
                        for i in range(n)]
    bars = ax.barh(y_pos, scores, color=colors, edgecolor='white', linewidth=0.5, height=0.7)
    ax.set_yticks(y_pos)
    ax.set_yticklabels(labels)
    ax.invert_yaxis()  # top rank at top
    ax.set_xlabel('Score')
    # Annotate bars
    for i, (bar, score) in enumerate(zip(bars, scores)):
        ax.text(bar.get_width() + 0.01, bar.get_y() + bar.get_height()/2,
                f'{score:.3f}', va='center', fontsize=8)
    return bars
```

### Grouped comparison bar
```python
def plot_grouped_comparison(ax, categories, series_dict, xlabel='', ylabel=''):
    """Grouped bar: series_dict = {label: [values], ...}"""
    n_groups = len(series_dict)
    n_cats = len(categories)
    x = np.arange(n_cats)
    width = 0.8 / n_groups
    colors = DEFAULT_COLORS[:n_groups]
    for i, (label, values) in enumerate(series_dict.items()):
        offset = (i - (n_groups - 1) / 2) * width
        ax.bar(x + offset, values, width, label=label, color=colors[i],
               edgecolor='white', linewidth=0.5)
    ax.set_xticks(x)
    ax.set_xticklabels(categories)
    ax.set_xlabel(xlabel)
    ax.set_ylabel(ylabel)
    ax.legend()
```

### Prediction vs actual
```python
def plot_prediction_vs_actual(ax, actual, predicted, xlabel='Actual', ylabel='Predicted'):
    """Scatter + identity line for prediction evaluation."""
    all_vals = np.concatenate([actual, predicted])
    lo, hi = all_vals.min(), all_vals.max()
    margin = (hi - lo) * 0.05
    ax.scatter(actual, predicted, c=PALETTE['primary'], s=30, alpha=0.7, edgecolors='white', linewidth=0.5)
    ax.plot([lo - margin, hi + margin], [lo - margin, hi + margin],
            '--', color=PALETTE['baseline_dark'], linewidth=1, alpha=0.7, label='y = x')
    ax.set_xlim(lo - margin, hi + margin)
    ax.set_ylim(lo - margin, hi + margin)
    ax.set_xlabel(xlabel)
    ax.set_ylabel(ylabel)
    ax.legend()
```

### Line trend with confidence band
```python
def plot_trend_with_confidence(ax, x, y_mean, y_lower=None, y_upper=None,
                                label='Prediction', color=None):
    """Line plot with optional confidence band. For time series and sensitivity."""
    color = color or PALETTE['primary']
    ax.plot(x, y_mean, color=color, linewidth=2, label=label)
    if y_lower is not None and y_upper is not None:
        ax.fill_between(x, y_lower, y_upper, color=color, alpha=0.15)
    ax.set_xlabel('Time / Index')
    ax.set_ylabel('Value')
    if label:
        ax.legend()
```

### Heatmap with annotations
```python
def plot_heatmap(ax, matrix, row_labels=None, col_labels=None, cmap='YlOrRd',
                 annotate=True, fmt='{:.2f}', cbar_label=''):
    """2D heatmap with optional cell annotations. Good for weight matrices, sensitivity results."""
    im = ax.imshow(matrix, cmap=cmap, aspect='auto')
    cbar = ax.figure.colorbar(im, ax=ax)
    if cbar_label:
        cbar.set_label(cbar_label)
    if row_labels:
        ax.set_yticks(range(len(row_labels)))
        ax.set_yticklabels(row_labels)
    if col_labels:
        ax.set_xticks(range(len(col_labels)))
        ax.set_xticklabels(col_labels, rotation=45, ha='right')
    if annotate:
        for i in range(matrix.shape[0]):
            for j in range(matrix.shape[1]):
                val = matrix[i, j]
                r, g, b, _ = plt.get_cmap(cmap)((val - matrix.min()) / (matrix.max() - matrix.min() + 1e-10))
                lum = 0.299 * r + 0.587 * g + 0.114 * b
                text_color = 'white' if lum < 0.5 else '#333333'
                ax.text(j, i, fmt.format(val), ha='center', va='center',
                        fontsize=7, color=text_color)
    ax.set_frame_on(False)
```

### Radar / polar chart (multi-method comparison)
```python
def plot_radar(ax, categories, series_dict, colors=None):
    """Radar chart for multi-criteria method comparison."""
    n = len(categories)
    angles = np.linspace(0, 2 * np.pi, n, endpoint=False).tolist()
    angles += angles[:1]  # close the polygon

    if colors is None:
        colors = DEFAULT_COLORS[:len(series_dict)]

    for (label, values), color in zip(series_dict.items(), colors):
        vals = list(values) + [values[0]]
        ax.fill(angles, vals, color=color, alpha=0.1)
        ax.plot(angles, vals, color=color, linewidth=2, label=label)
        ax.scatter(angles[:-1], values, color=color, s=30, zorder=5)

    ax.set_xticks(angles[:-1])
    ax.set_xticklabels(categories, fontsize=8)
    ax.set_ylim(0, 1.0)
    ax.legend(loc='upper right', bbox_to_anchor=(1.3, 1.1))
```

### Sensitivity / tornado plot
```python
def plot_sensitivity_lines(ax, param_values, result_matrix, param_name='Parameter',
                            labels=None, colors=None):
    """Sensitivity curves: x=parameter value, y=result. One line per output metric."""
    n_lines = result_matrix.shape[1] if result_matrix.ndim > 1 else 1
    if result_matrix.ndim == 1:
        result_matrix = result_matrix.reshape(-1, 1)
    if colors is None:
        colors = DEFAULT_COLORS[:n_lines]
    if labels is None:
        labels = [f'Metric {i+1}' for i in range(n_lines)]
    for j in range(n_lines):
        ax.plot(param_values, result_matrix[:, j], color=colors[j],
                linewidth=2, marker='o', markersize=5, label=labels[j])
    ax.set_xlabel(param_name)
    ax.set_ylabel('Result')
    ax.legend()
```

## Multi-panel information architecture

Follow the three-level progressive complexity pattern adapted for math modeling:

| Level | Question | Panel Type |
|-------|----------|------------|
| Overview | "What is the big picture?" | Workflow diagram, schematic, overall ranking |
| Main Evidence | "What does the model produce?" | Ranking bar, prediction plot, allocation chart |
| Validation | "Is the result reliable?" | Sensitivity line, baseline comparison, error plot |

### Anti-redundancy checklist

Before finalizing, verify no two panels answer the same question:
- [ ] Panel `a` and panel `b` show different information (not the same data in two chart types).
- [ ] If two panels compare methods, they compare on different criteria.
- [ ] Sensitivity panels each cover a different parameter or assumption.
- [ ] Remove any panel that, if covered, would not weaken the conclusion.

## Output rules

- **Primary format is SVG** (editable text, scalable). Always save `.svg` first.
- **Secondary is PNG** at ≥300 dpi for quick preview and paper insertion.
- For final paper, copy the polished figure to `paper/figures/`.
- For experiment rounds, save to `results/Qx/experiments/roundN/figures/`.
- For robustness figures, save to `robustness/Qx/figures/`.
- Always `plt.close(fig)` after saving.

## Headless rendering

```python
import matplotlib
matplotlib.use('Agg')       # non-interactive backend — always first
import matplotlib.pyplot as plt
```

## Reproducibility

- Fix random seeds for any stochastic element (bootstrap, jitter, sampling).
- Record data source paths in code comments.
- Save the exact script that generated each figure alongside the figure.
- Use `run_summary.json` to record figure generation details: script, data source, parameters.

## Figure QA checklist

Before handing off a figure:
- [ ] Core claim is clear and the figure defends it.
- [ ] Figure type from the math modeling taxonomy is correctly identified.
- [ ] Every panel carries unique evidence (anti-redundancy passed).
- [ ] SVG saved with `svg.fonttype = 'none'` (editable text).
- [ ] Font is sans-serif (Arial/Helvetica/DejaVu Sans fallback stack).
- [ ] Right and top spines are off; `legend.frameon = False`.
- [ ] Colors are from the defined palette and consistent across panels.
- [ ] Same method/concept has the same color in every panel.
- [ ] Y-limits are tight to data range.
- [ ] Panel labels are bold lowercase (a, b, c).
- [ ] Statistics, sample size, and error bar definitions are documented.
- [ ] Source data path is recorded.
- [ ] Figure is assigned to a paper section (or marked as diagnostic/internal).
- [ ] `plt.close(fig)` after save.

## When to use this skill

Use when:
- Creating new figures for a math modeling contest paper.
- Revising existing figures to improve quality or logic.
- The user asks: "make a ranking chart", "plot the prediction", "create a comparison figure", "generate paper figures", "make publication-quality charts".
- Multi-panel figures are needed that combine different evidence types.
- The workflow reaches `figure-table-planner` and code needs to implement the planned figures.

## When NOT to use

- Interactive/dashboard plots (Plotly, Bokeh, etc.).
- EDA-only quick plots without a publication target — use quick matplotlib directly.
- 3D rendering, GIS, or non-matplotlib illustration tooling.
- The figure-table-planner has not yet planned the figures (coordinate with the planner).

## Related files

| File | Purpose |
|------|---------|
| [references/chart-patterns.md](references/chart-patterns.md) | Extended chart pattern gallery with full code examples |
| [references/color-systems.md](references/color-systems.md) | Detailed color theory for math modeling figures |
| [references/layout-guide.md](references/layout-guide.md) | Multi-panel layout strategies and common patterns |

## Rules

- Start from the figure contract — do not code before defining the claim.
- Use Python/matplotlib unless the project's implementation target is MATLAB.
- For MATLAB users: adapt the logic to MATLAB's `plot`, `bar`, `heatmap`, etc. The conceptual rules (claim-first, panel logic, color consistency) still apply.
- Primary output is always SVG + PNG.
- Every figure must have a source data path.
- Colors must be consistent: same method = same color across all panels.
- Do not fabricate data or create figures from invented numbers.
- Do not create decorative figures that don't support a specific claim.
- Close every figure after saving.
- For multi-panel figures, the hero panel should be visually dominant.
