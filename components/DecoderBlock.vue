<script setup>
import DecoderBlockDiagram from './DecoderBlockDiagram.vue'
</script>

<template>
  <div class="decoder-block">
    <DecoderBlockDiagram />

    <div class="explanation">
      <section class="explanation-step">
        <span class="step attention-text" aria-hidden="true">1</span>
        <div>
          <h2 class="heading">Gather context</h2>
          <p>For each position, each attention head gathers context with its own Q, K, and V. Their outputs are then combined.</p>
        </div>
      </section>
      <section class="explanation-step">
        <span class="step mlp-text" aria-hidden="true">2</span>
        <div>
          <h2 class="heading">Transform features</h2>
          <p>Apply the same MLP to each position in parallel.</p>
        </div>
      </section>
      <div class="block-notes">
        <p>To <strong>normalize</strong>, subtract the mean and divide by the standard deviation. This keeps feature scales under control.</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.decoder-block {
  display: grid;
  grid-template-columns: 400px minmax(0, 1fr);
  gap: 22px;
  color: var(--diagram-ink);
}

.explanation {
  padding-top: 46px;
}

.explanation-step {
  display: grid;
  grid-template-columns: 25px minmax(0, 1fr);
  gap: 14px;
}

.explanation-step + .explanation-step {
  margin-top: 34px;
}

.explanation .heading {
  margin: 0;
  padding: 0;
  font-size: 25px;
  line-height: 30px;
  font-weight: 600;
}

.explanation-step p {
  margin: 14px 0 0;
  font-size: 18px;
  line-height: 26px;
}

.block-notes {
  margin-top: 16px;
  padding-top: 14px;
  border-top: 1px solid color-mix(in oklch, var(--diagram-muted) 35%, transparent);
}

.block-notes p {
  margin: 0;
  font-size: 15px;
  line-height: 26px;
}

.block-notes strong {
  font-weight: 600;
}

.block-notes .dimensions {
  margin-top: 3px;
  color: var(--diagram-muted);
}

.step {
  line-height: 30px;
  font-size: 28px;
  font-weight: 600;
}

.attention-text {
  color: var(--diagram-attention);
}

.mlp-text {
  color: var(--diagram-mlp);
}
</style>
