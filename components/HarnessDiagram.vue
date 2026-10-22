<template>
  <svg class="harness-diagram" viewBox="0 0 880 330" role="img" aria-labelledby="harness-title harness-desc">
    <title id="harness-title">A harness connects a model to tools and feeds results back into context</title>
    <desc id="harness-desc">Inside application code, system instructions, conversation history, observations, and tool schemas form the model's context. The model requests a tool call. The harness validates and executes it against files, APIs, or a terminal. Results return to context for the next model call. The harness manages budgets and stopping and returns the final answer.</desc>
    <defs>
      <marker id="harness-arrow" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="5" markerHeight="5" orient="auto"><path d="M0 0 L10 5 L0 10z" fill="currentColor" /></marker>
    </defs>
    <rect x="8" y="16" width="666" height="292" rx="10" class="boundary" />
    <text x="30" y="49" class="title">Harness</text>

    <g fill="none" stroke="currentColor" stroke-width="2" marker-end="url(#harness-arrow)" class="wires">
      <path d="M245 153 H306" />
      <path d="M436 153 H503" />
      <path d="M652 153 H730" />
      <path d="M796 211 V258 H138 V216" class="feedback" />
    </g>
    <rect x="30" y="86" width="215" height="130" rx="6" class="context" />
    <text x="47" y="113" class="label">Assemble context</text>
    <text x="47" y="140" class="detail">System instructions</text>
    <text x="47" y="163" class="detail">Chat history</text>
    <text x="47" y="186" class="detail">Tool schemas</text>
    <text x="47" y="209" class="detail">User input</text>

    <rect x="309" y="115" width="127" height="77" rx="6" class="model" />
    <text x="372" y="148" text-anchor="middle" class="title">Model</text>
    <text x="372" y="173" text-anchor="middle" class="detail">choose next step</text>

    <text x="472" y="135" text-anchor="middle" class="small">tool call</text>
    <rect x="506" y="115" width="146" height="77" rx="6" class="tool" />
    <text x="579" y="147" text-anchor="middle" class="label">Execute tool</text>
    <text x="579" y="173" text-anchor="middle" class="detail">validate + run</text>

    <text x="796" y="93" text-anchor="middle" class="label">Outside world</text>
    <path d="M744 112 H732 V211 H744 M848 112 H860 V211 H848" class="bracket" />
    <text x="796" y="139" text-anchor="middle">Files</text>
    <text x="796" y="168" text-anchor="middle">APIs</text>
    <text x="796" y="197" text-anchor="middle">Terminal</text>

    <text x="432" y="244" text-anchor="middle" class="feedback-label">Results become context for the next step</text>
    <text x="30" y="291" class="small muted">Also manages execution limits, budgets, stopping, and UI</text>
  </svg>
</template>

<style scoped>
.harness-diagram {
  display: block;
  width: 100%;
  height: 330px;
  color: var(--diagram-ink);
}

text {
  fill: currentColor;
  font-family: inherit;
  font-size: 18px;
}

.title {
  font-size: 24px;
  font-weight: 600;
}

.label {
  font-size: 20px;
  font-weight: 600;
}

.detail {
  font-size: 16px;
}

.small {
  font-size: 15px;
}

.muted, .wires {
  color: var(--diagram-muted);
}

.boundary {
  fill: none;
  stroke: var(--diagram-muted);
  stroke-width: 1.2;
  stroke-dasharray: 6 5;
}

.context {
  fill: var(--diagram-attention-fill);
  stroke: var(--diagram-attention);
  stroke-width: 1.5;
}

.model {
  fill: var(--diagram-surface);
  stroke: var(--diagram-muted);
  stroke-width: 1.5;
}

.tool {
  fill: var(--diagram-mlp-fill);
  stroke: var(--diagram-mlp);
  stroke-width: 1.5;
}

.bracket {
  fill: none;
  stroke: var(--diagram-muted);
  stroke-width: 1.5;
}

.feedback, .feedback-label {
  color: var(--diagram-attention);
}

.feedback-label {
  font-size: 16px;
}
</style>
