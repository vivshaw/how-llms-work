<template>
  <svg class="decoder-stack" viewBox="0 0 880 174" role="img" aria-labelledby="stack-title stack-desc">
    <title id="stack-title">From token vectors through decoder blocks to vocabulary scores</title>
    <desc id="stack-desc">Token vectors pass through blocks one, two, up to N, all maintaining shape T by d. A final normalization and vocabulary projection produce scores of shape T by V. For generation, use the final position's scores.</desc>
    <defs><marker id="stack-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5" markerHeight="5" orient="auto"><path d="M0 0 L10 5 L0 10z" fill="currentColor" /></marker></defs>
    <g fill="none" stroke="currentColor" stroke-width="2" marker-end="url(#stack-arrow)" class="wires">
      <path d="M 119 78 H 148" /><path d="M 263 78 H 285" /><path d="M 400 78 H 470" /><path d="M 585 78 H 618" /><path d="M 746 78 H 777" />
    </g>
    <g text-anchor="middle">
      <text x="57" y="65">Token</text><text x="57" y="88">vectors</text><text x="57" y="117" class="muted">T × d</text>
      <g v-for="block in [{ x: 151, label: 'Block 1' }, { x: 288, label: 'Block 2' }, { x: 473, label: 'Block N' }]" :key="block.label">
        <rect :x="block.x" y="28" width="112" height="99" rx="6" class="block" />
        <text :x="block.x + 56" y="53" class="block-title">{{ block.label }}</text>
        <text :x="block.x + 56" y="80" class="attention-text">Attention</text>
        <text :x="block.x + 56" y="107" class="mlp-text">MLP</text>
      </g>
      <text x="436" y="59" class="ellipsis">…</text>
      <text x="368" y="159" class="muted">Same structure, different learned weights in each block</text>
      <rect x="621" y="28" width="125" height="99" rx="6" class="block" />
      <text x="683" y="56">Final norm</text><text x="683" y="80">+ vocabulary</text><text x="683" y="104">projection</text>
      <text x="827" y="65">Vocabulary</text><text x="827" y="88">scores</text><text x="827" y="117" class="muted">T × V</text>
    </g>
  </svg>
</template>

<style scoped>
.decoder-stack {
  width: 100%;
  height: 174px;
  color: var(--diagram-ink);
}

text {
  fill: currentColor;
  font-family: inherit;
  font-size: 17px;
}

.muted, .wires {
  color: var(--diagram-muted);
  fill: currentColor;
}

.muted {
  font-size: 15px;
}

.block {
  fill: var(--diagram-surface);
  stroke: var(--diagram-muted);
  stroke-width: 1.2;
}

.block-title {
  font-weight: 700;
  font-size: 19px;
}

.attention-text {
  fill: var(--diagram-attention);
}

.mlp-text {
  fill: var(--diagram-mlp);
}

.ellipsis {
  font-size: 29px;
}
</style>
