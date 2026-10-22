<script setup>
const starryNightUrl = `${import.meta.env.BASE_URL}images/starry-night.jpg`
</script>

<template>
  <div class="network-types">
    <section>
      <h2>Multi-layer perceptron</h2>
      <svg viewBox="0 0 380 110" role="img" aria-label="Dense connections between successive layers of neurons">
        <g class="connections">
          <g v-for="x in [65, 185]" :key="x"><g v-for="y in [22, 55, 88]" :key="y"><line v-for="next in [22, 55, 88]" :key="next" :x1="x" :y1="y" :x2="x + 120" :y2="next" /></g></g>
        </g>
        <g v-for="x in [65, 185, 305]" :key="x"><circle v-for="y in [22, 55, 88]" :key="y" :cx="x" :cy="y" r="10" class="teal" /></g>
      </svg>
      <p>Each neuron mixes inputs from the previous layer.</p>
    </section>
    <section>
      <h2>Convolutional network</h2>
      <svg viewBox="0 0 380 110" role="img" aria-label="A 3 by 3 filter scans a 4 by 7 input with stride one and no padding, producing a 2 by 5 feature map">
        <image opacity="0.8" :href="starryNightUrl" x="30" y="10" width="159" height="90" preserveAspectRatio="xMidYMid slice" />
        <g v-for="row in 4" :key="row"><rect v-for="col in 7" :key="col" :x="30 + (col - 1) * 23" :y="10 + (row - 1) * 23" width="21" height="21" class="cell" /></g>
        <rect x="29" y="9" width="69" height="69" rx="3" class="filter" />
        <path d="M109 41 H168 M159 33 L168 41 L159 49" class="scan-arrow" />
        <path d="M207 55 H244 M235 47 L244 55 L235 63" class="arrow" />
        <g v-for="(row, r) in [[0, 2, 5, 1, 0], [1, 4, 8, 3, 1]]" :key="r"><g v-for="(value, c) in row" :key="c"><rect :x="252 + c * 23" :y="32 + r * 23" width="21" height="21" class="amber" /><text :x="262.5 + c * 23" :y="47 + r * 23" class="map-value">{{ value }}</text></g></g>
        <text x="308" y="99" class="small">Example feature map</text>
      </svg>
      <p>A learned filter scans across local neighborhoods.</p>
    </section>
    <section>
      <h2>Recurrent network</h2>
      <svg viewBox="0 0 380 110" role="img" aria-label="A layered recurrent network with feedback loops representing hidden state from the previous sequence step">
        <g class="connections">
          <g v-for="(x, i) in [65, 145, 235]" :key="x"><g v-for="y in [30, 65, 100]" :key="y"><line v-for="nextY in [30, 65, 100]" :key="nextY" :x1="x" :y1="y" :x2="[145, 235, 315][i]" :y2="nextY" /></g></g>
        </g>
        <g v-for="x in [145, 235]" :key="x">
          <path v-for="y in [30, 65, 100]" :key="y" :d="`M${x + 7} ${y - 7} C${x + 24} ${y - 35}, ${x - 24} ${y - 35}, ${x - 7} ${y - 7} m1 -8 l-1 8 l-7 -4`" class="arrow" />
        </g>
        <g v-for="x in [65, 145, 235, 315]" :key="x"><circle v-for="y in [30, 65, 100]" :key="y" :cx="x" :cy="y" r="9" :class="x === 315 ? 'amber' : 'teal'" /></g>
      </svg>
      <p>Each step also takes feedback from previous step.</p>
    </section>
    <section>
      <h2>Transformer</h2>
      <svg viewBox="0 0 380 110" role="img" aria-label="Causal attention lets each position gather information from itself and earlier positions">
        <defs><marker id="types-attention-arrow" viewBox="0 0 10 10" refX="10" refY="5" markerWidth="6" markerHeight="6" orient="auto"><path d="M0 0 L10 5 L0 10z" fill="var(--diagram-muted)" stroke="none" /></marker></defs>
        <g class="connections" marker-end="url(#types-attention-arrow)"><g v-for="(x, i) in [55, 145, 235, 325]" :key="x"><line v-for="from in [55, 145, 235, 325].slice(0, i + 1)" :key="from" :x1="from" y1="78" :x2="x" y2="34" /></g></g>
        <g v-for="(x, i) in [55, 145, 235, 325]" :key="x">
          <rect :x="x - 23" y="8" width="46" height="24" rx="4" class="teal" />
          <rect :x="x - 23" y="78" width="46" height="24" rx="4" class="amber" />
          <text :x="x" y="96">{{ ['The', 'cat', 'sat', 'on'][i] }}</text>
        </g>
      </svg>
      <p>Use attention to mix info across token positions.</p>
    </section>
  </div>
  <div class="depth-note"><strong>"Deep network" just means "many learned layers".</strong> Any of these architectures can be deep.</div>
</template>

<style scoped>
.network-types {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 22px 34px;
  color: var(--diagram-ink);
}

.network-types h2 {
  font-size: 23px;
  line-height: 1.25;
  font-weight: 600;
  margin: 0 0 7px;
}

.network-types svg {
  display: block;
  width: 100%;
  height: 110px;
  overflow:visible;
}

.network-types p {
  font-size: 17px;
  line-height: 1.35;
  margin: 6px 0 0;
}

text {
  font-family: inherit;
  font-size: 17px;
  text-anchor: middle;
  fill: var(--diagram-ink);
}

.map-value {
  font-size: 13px;
  font-weight: 600;
}

.small {
  font-size: 13px;
  fill: var(--diagram-muted);
}

.connections {
  stroke: var(--diagram-muted);
  stroke-width: 1.3;
  stroke-opacity: 0.55;
}

.teal {
  fill: var(--diagram-attention-fill);
  stroke: var(--diagram-attention);
  stroke-width: 1.5;
}

.amber {
  fill: var(--diagram-mlp-fill);
  stroke: var(--diagram-mlp);
  stroke-width: 1.5;
}

.cell {
  fill: none;
  stroke: oklch(0.94 0.005 240);
  stroke-width: 0.8;
  stroke-opacity: 0.65;
}

.filter {
  fill: var(--diagram-attention-fill);
  fill-opacity: 0.18;
  stroke: var(--diagram-attention);
  stroke-opacity: 1;
  stroke-width: 3;
}

.scan-arrow {
  fill: none;
  stroke: var(--diagram-attention);
  stroke-opacity: 1;
  stroke-width: 3;
  stroke-linecap: round;
  stroke-linejoin: round;
}

.arrow {
  fill: none;
  stroke: var(--diagram-muted);
  stroke-width: 1.8;
}

.depth-note {
  margin-top: 16px;
  padding-top: 12px;
  border-top: 1px solid var(--diagram-muted);
  font-size: 18px;
  line-height: 1.4;
}

.depth-note strong {
  font-weight: 600;
}
</style>
