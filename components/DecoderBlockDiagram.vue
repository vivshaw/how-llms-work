<template>
  <svg class="decoder-diagram" viewBox="0 0 400 438" role="img" aria-labelledby="decoder-title decoder-desc">
    <title id="decoder-title">One decoder block: gather context, then transform features</title>
    <desc id="decoder-desc">Input vectors flow through normalization, then branch into three illustrative parallel causal attention heads with different learned projections. Their outputs are concatenated and projected, then added to a bypass of the input. A second normalization and MLP produce an update added through another residual connection. Output has the same sequence length and feature width. This is a pre-normalization decoder block.</desc>
    <defs>
      <marker id="decoder-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5" markerHeight="5" orient="auto-start-reverse"><path d="M 0 0 L 10 5 L 0 10 z" fill="currentColor" /></marker>
    </defs>
    <g class="wires" fill="none" stroke="currentColor" stroke-width="2" marker-end="url(#decoder-arrow)">
      <path d="M 234 25 V 43" />
      <path d="M 234 74 V 88" />
      <path d="M 234 212 V 224" />
      <path d="M 234 260 V 274" />
      <path d="M 234 302 V 314" />
      <path d="M 234 355 V 367" />
      <path d="M 234 403 V 414" />
      <path d="M 234 32 H 66 V 242 H 214" />
      <path d="M 234 267 H 66 V 385 H 214" />
    </g>
    <g class="labels" text-anchor="middle">
      <text x="234" y="18" class="endpoint">Input vectors</text>
      <rect x="126" y="44" width="216" height="30" rx="5" class="norm" />
      <text x="234" y="65" class="small">Normalize</text>
      <rect x="86" y="90" width="296" height="122" rx="6" class="attention" />
      <text x="234" y="112" class="module">Causal self-attention</text>
      <g class="head-wires" fill="none" stroke="currentColor" stroke-width="1.4" marker-end="url(#decoder-arrow)">
        <path d="M 234 118 V 124 H 138 V 132" />
        <path d="M 234 118 V 132" />
        <path d="M 234 124 H 330 V 132" />
        <path d="M 138 158 V 167 H 234 V 178" />
        <path d="M 234 158 V 178" />
        <path d="M 330 158 V 167 H 234" marker-end="none" />
      </g>
      <g v-for="(x, index) in [96, 192, 288]" :key="x">
        <rect :x="x" y="133" width="84" height="25" rx="4" class="head" />
        <text :x="x + 42" y="151" class="head-label">Head {{ index + 1 }}</text>
      </g>
      <rect x="103" y="179" width="262" height="25" rx="4" class="head" />
      <text x="234" y="197" class="small">Combine the outputs</text>
      <circle cx="234" cy="242" r="18" class="sum" /><text x="234" y="249" class="plus">+</text>
      <rect x="126" y="275" width="216" height="27" rx="5" class="norm" />
      <text x="234" y="294" class="small">Normalize</text>
      <rect x="106" y="316" width="256" height="39" rx="6" class="mlp" />
      <text x="234" y="334" class="module">Multi-layer perceptron</text>
      <text x="234" y="350" class="small">(Shallow, 2 layers is plenty)</text>
      <circle cx="234" cy="385" r="18" class="sum" /><text x="234" y="392" class="plus">+</text>
      <text x="234" y="432" class="endpoint">Updated vectors</text>
      <text x="50" y="140" transform="rotate(-90 50 140)" class="small muted">carry input forward</text>
      <text x="50" y="330" transform="rotate(-90 50 330)" class="small muted">carry input forward</text>
    </g>
  </svg>
</template>

<style scoped>
.decoder-diagram {
  display: block;
  width: 100%;
  height: 414px;
  color: var(--diagram-ink);
}

text {
  fill: currentColor;
  font-family: inherit;
  font-size: 18px;
}

.endpoint {
  font-size: 18px;
  font-weight: 600;
}

.small {
  font-size: 15px;
}

.module {
  font-size: 19px;
  font-weight: 600;
}

.plus {
  font-size: 25px;
}

.muted, .wires {
  color: var(--diagram-muted);
}

.norm, .sum {
  fill: var(--diagram-surface);
  stroke: var(--diagram-muted);
  stroke-width: 1.3;
}

.attention {
  fill: var(--diagram-attention-fill);
  stroke: var(--diagram-attention);
  stroke-width: 1.5;
}

.head {
  fill: var(--diagram-surface);
  stroke: var(--diagram-attention);
  stroke-width: 1;
}

.head-label {
  font-size: 15px;
  font-weight: 600;
}

.head-wires {
  color: var(--diagram-attention);
}

.mlp {
  fill: var(--diagram-mlp-fill);
  stroke: var(--diagram-mlp);
  stroke-width: 1.5;
}
</style>
