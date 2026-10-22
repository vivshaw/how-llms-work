<script setup>
const layers = [
  { x: 42, ys: [91, 135, 179], name: 'Inputs', detail: 'x', kind: 'input' },
  { x: 196, ys: [69, 113, 157, 201], name: 'Hidden layer 1', detail: 'h₁ (ReLU)', kind: 'hidden' },
  { x: 356, ys: [69, 113, 157, 201], name: 'Hidden layer 2', detail: 'h₂ (ReLU)', kind: 'hidden' },
  { x: 510, ys: [113, 157], name: 'Output layer', detail: 'y (Linear)', kind: 'output' },
]
</script>

<template>
  <svg class="network-diagram" viewBox="0 0 575 270" role="img" aria-labelledby="network-title network-desc">
    <title id="network-title">A fully connected neural network with three learned layers</title>
    <desc id="network-desc">Three input values connect to four neurons in hidden layer one, then four neurons in hidden layer two, then two output neurons. Every neuron receives all values from the preceding layer, with its own weights and bias. Hidden neurons apply ReLU; output neurons use a linear transformation. Widths are illustrative. Inputs are not a learned layer.</desc>
    <g class="connections">
      <g v-for="(layer, i) in layers.slice(0, -1)" :key="i">
        <g v-for="y in layer.ys" :key="y">
          <line v-for="nextY in layers[i + 1].ys" :key="nextY" :x1="layer.x + 12" :y1="y" :x2="layers[i + 1].x - 18" :y2="nextY" />
        </g>
      </g>
    </g>
    <g v-for="layer in layers" :key="layer.name" text-anchor="middle">
      <text :x="layer.x" y="23" class="heading">{{ layer.name }}</text>
      <circle v-for="y in layer.ys" :key="y" :cx="layer.x" :cy="y" :r="layer.kind === 'input' ? 12 : 18" :class="`neuron-${layer.kind}`" />
      <text :x="layer.x" y="241" class="detail">{{ layer.detail }}</text>
    </g>
  </svg>
</template>

<style scoped>
.network-diagram {
  display: block;
  width: 100%;
  height: 250px;
  color: var(--diagram-ink);
}

text {
  fill: currentColor;
  font-family: inherit;
}

.heading {
  font-size: 17px;
  font-weight: 600;
}

.detail {
  font-size: 18px;
  font-weight: 600;
}

.connections {
  stroke: var(--diagram-muted);
  stroke-width: 1.1;
  stroke-opacity: 0.45;
}

.neuron-input {
  fill: var(--diagram-surface);
  stroke: var(--diagram-muted);
  stroke-width: 1.5;
}

.neuron-hidden {
  fill: var(--diagram-attention-fill);
  stroke: var(--diagram-attention);
  stroke-width: 1.5;
}

.neuron-output {
  fill: var(--diagram-mlp-fill);
  stroke: var(--diagram-mlp);
  stroke-width: 1.5;
}
</style>
