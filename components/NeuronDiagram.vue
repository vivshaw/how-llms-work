<template>
  <svg class="neuron-diagram" viewBox="0 0 880 210" role="img" aria-labelledby="neuron-title neuron-desc">
    <title id="neuron-title">A neuron multiplies inputs by weights, adds them and a bias, then applies ReLU</title>
    <desc id="neuron-desc">Three inputs x1, x2, and x3 are multiplied by learned weights w1, w2, and w3. Their products are summed, a learned bias b is added, and ReLU returns the larger of zero and that result, producing one output y.</desc>
    <defs><marker id="neuron-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5" markerHeight="5" orient="auto"><path d="M0 0 L10 5 L0 10z" fill="currentColor" /></marker></defs>
    <rect x="130" y="3" width="630" height="204" rx="9" class="neuron-boundary" />
    <g class="wires" fill="none" stroke="currentColor" stroke-width="2" marker-end="url(#neuron-arrow)">
      <path d="M90 60 H166" /><path d="M90 110 H166" /><path d="M90 160 H166" />
      <path d="M250 60 L326 95" /><path d="M250 110 H322" /><path d="M250 160 L326 125" />
      <path d="M378 110 H449" /><path d="M501 110 H578" /><path d="M730 110 H800" />
      <path d="M475 57 V82" />
    </g>
    <g text-anchor="middle">
      <text x="65" y="23" class="caption">Inputs</text>
      <text x="210" y="23" class="caption">Weights</text>
      <g v-for="(row, i) in [{ y: 60, x: 'x₁', w: 'w₁' }, { y: 110, x: 'x₂', w: 'w₂' }, { y: 160, x: 'x₃', w: 'w₃' }]" :key="i">
        <text x="65" :y="row.y + 7" class="symbol">{{ row.x }}</text>
        <rect x="170" :y="row.y - 18" width="80" height="36" rx="5" class="weight" />
        <text x="210" :y="row.y + 7" class="symbol">× {{ row.w }}</text>
      </g>
      <circle cx="350" cy="110" r="27" class="neutral" />
      <text x="350" y="119" class="sum">Σ</text>
      <text x="350" y="170" class="caption">Add products</text>
      <rect x="442" y="21" width="66" height="36" rx="5" class="weight" />
      <text x="475" y="47" class="symbol">b</text>
      <text x="532" y="46" class="caption">Bias</text>
      <circle cx="475" cy="110" r="25" class="neutral" />
      <text x="475" y="118" class="sum">+</text>
      <text x="539" y="98" class="symbol">z</text>
      <rect x="580" y="76" width="150" height="68" rx="6" class="activation" />
      <text x="655" y="104" class="label">ReLU</text>
      <text x="655" y="129" class="caption">max(0, z)</text>
      <text x="655" y="170" class="caption">Activation</text>
      <text x="823" y="118" class="symbol">y</text>
      <text x="823" y="170" class="caption">Output</text>
    </g>
  </svg>
</template>

<style scoped>
.neuron-diagram {
  display: block;
  width: 100%;
  height: 200px;
  color: var(--diagram-ink);
}

text {
  fill: currentColor;
  font-family: inherit;
}

.symbol {
  font-size: 24px;
  font-weight: 600;
}

.label {
  font-size: 23px;
  font-weight: 600;
}

.sum {
  font-size: 29px;
}

.caption {
  font-size: 17px;
}

.note {
  font-size: 15px;
  fill: var(--diagram-muted);
}

.wires {
  color: var(--diagram-muted);
}

.neuron-boundary {
  fill: none;
  stroke: var(--diagram-muted);
  stroke-width: 1.2;
  stroke-opacity: 0.55;
}

.neutral {
  fill: var(--diagram-surface);
  stroke: var(--diagram-muted);
  stroke-width: 1.5;
}

.weight {
  fill: var(--diagram-attention-fill);
  stroke: var(--diagram-attention);
  stroke-width: 1.5;
}

.activation {
  fill: var(--diagram-mlp-fill);
  stroke: var(--diagram-mlp);
  stroke-width: 1.5;
}
</style>
