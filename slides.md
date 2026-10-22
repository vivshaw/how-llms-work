---
theme: default
title: How LLMs work
info: |
  From linear equations, to reasoning and tool-using systems
class: text-center
transition: none
mdc: true
drawings:
  persist: false
---

# How LLMs work

From linear equations, to reasoning and tool-using systems

---

# Why should we care?

- Understanding what a model itself does vs. the code around it
- Reason better about failures when you observe them
- Make better choices about how to prompt and finetune
- We're engineers! We love to peek under the hood!

**Example:** A test failed. You asked Claude to fix it. And it did! What happened in this scenario, step by step? Can you answer?

<!--
- Relate this to the "When you type a URL in the browser and hit "Enter"" interview Q
- By the time you';'re done with this talk, you'll be able to answer
-->

---

# Moving parts

| Part | What it does |
| --- | --- |
| Model | Maps a token sequence to predictions |
| Tokenizer & Embedding | Convert text into mathematical objects the model can work with |
| Training | Adjusts the model from examples and rewards to improve it |
| Harness | Supplies context and runs tools |

I will build up your intuitions about each piece, and connect them along the way

<!--
- Note that it won't be a linear journey. We'll skip around a bit based on narrative flow. Don't worry, it'll all link up
-->

---
class: neuron-slide
---

# Neuron: A simple function

$$
y = \mathrm{ReLU}(w_1x_1 + w_2x_2 + \cdots + w_nx_n + b)
$$

<NeuronDiagram />

- Inputs: numbers, collected into a vector.
- Parameters: some weights and a bias.
- Activation: a nonlinear function, such as a sigmoid, or "rectified linear units" (ReLU: $\max(0, z)$).

This is just a normal linear equation plus a little something extra!

<!--
- Here's where the math starts. Don't be scared! I'm gonna show it to you three ways: math way, picture way, and plain English
- This is basically a linear equation! y = mx + b
- Nonlinearity is a secret tool that will help us later.
-->

---
class: network-slide
---

# Neural network: Composition of neurons

<div class="network-overview">
<NetworkDiagram />
<div>

$$
\begin{aligned}
h_1 &= \mathrm{ReLU}(W_1x + b_1) \\
h_2 &= \mathrm{ReLU}(W_2h_1 + b_2) \\
y &= W_3h_2 + b_3
\end{aligned}
$$

</div>
</div>

- A neural network is a stack of connected layers of neurons.
- We use linear algebra to compute many neurons simultaneously.
- Without nonlinearities, "stacked affine layers collapse to one affine map". Huh?
  - In plain English: a stack of linear layers can only learn a linear function! That's the same as a single layer!
  - The nonlinear bit lets your stacked layers learn more complex patterns.

<!--
- Once again: math way, and the picture way
- In a neural netowrk, we basically make a bunch of layers of neurons, and stack them up, connecting the layers
- Here's where the nonlinear activations come in.
-->

---

# Some types of neural networks

<NetworkTypes />

<!--
- Multi-layer perceptron is just the layered network we've already seen
- Convolutional networks, great for images
- Recurrent networks basically give the model a "short-term memory" to work with sequenced data, like text or timeseries
- Transformers are what modern LLMs are built on. They're even better for text. We'll talk about them in more depth later!
-->

---

# What LMs do: Predict the next word

$$
p_\theta(t_{i+1}\mid t_1,\ldots,t_i)
$$

Given some text, produce a probability distribution over all known words for what comes immediately next.

```text
Prefix:      The capital of France is
Next token:  ?
```

...but how do we get from _words_, to _numbers_ that a neural network can work with?

<!--
- Prob for _every_ word: Paris, Moscow, Football, etc.
- Ideally, the best word or words has the highest probability
- One teensy weensy itty bitty problem. Our task is about words. But neural networks work with numbers!
-->

---

# Tokenizing: Text must become numbers!

```text
text => tokenizer => token IDs => embedding => parameter vectors
```

- **Tokenizer:** converts strigns of text into vectors of numerical IDs from a fixed vocabulary.
- **Token ID:** a number representing a text fragment. (Purely an index, not a meaningful scalar.)
- **Embedding:** a learned vector of parameters for each vocabulary entry.

The tokenizer and the model must use the same vocabulary and IDs.

---

# Tokenizing: How do we chunk up the text?

| Unit | Advantage | Tradeoff |
| --- | --- | --- |
| Character or byte | Small units, broad coverage (no unfamiliar words) | Long sequences, model must learn word-level patterns |
| Whole word | Shorter sequences, familiar tokens | Huge vocabulary, unfamiliar words are a problem, model must learn grammatical connections (e.g. between "run" and "running") |
| Byte merges | Balance vocab size and sequence length, reuse common features ("-ing", "-s"), broad coverage. | Boundaries can surprise us: pieces reflect frequency, not grammar or meaning |

<!--
- OK, time to learn some natural language processing!
- We could break it up by individual characters. This would let us construct any text we want! But it would carry zero semantic information. Imagine how much harder it would be to learn English if you could only see letters, not words?
- We could break it up by words. A vocab with pre-baked English words should help, right? Well... there are so many words out there the vocabulary would be unworkably huge. And what about encountering unfamiliar words?
- Enter byte merges! These mix the strengths of each of these: you can represent whole words where useful, but individual letters where needed. And you can capture some reusable pieced like "-ing", "-s", "-ed".
-->

---

# Byte-pair encoding: building a tokenizer

Here's how we'll break text into reusable pieces:

1. Start with small symbols, like individual bytes
2. Merge the most frequent adjacent pair in your text corpus
3. Save that merge to an ordered list
4. Repeat steps 2 & 3 until hitting a stop condition (maximum vocab size, minimum pair frequency, etc.)

(Practical tokenizers usually apply boundary rules too, like splitting the corpus on spaces and/or punctuation)

For an example of a real tokenizer: OpenAI's [tiktoken](https://github.com/openai/tiktoken) is OSS!

<!--
- BPE is super common as a tokenizer. Almost all frontier models use it.
- Real tokenizers generally do some pre-chunking with boundary rules, like first splitting on spaces or punctuation, before applying the merges
-->

---

# Byte-pair encoding: tokenizing text

Simply apply the list of merges in order. Example:

```text
Actual GPT-2 rules:
  #1     ␠ + t     => ␠t
  #3     h + e     => he
  #7     ␠t + he   => ␠the

Encoding " the":  ␠ t h e => ␠t h e => ␠t he => ␠the
```

[Source: GPT-2's published merge rules](https://huggingface.co/openai-community/gpt2/blob/main/merges.txt).

<!--
- This is not a hypothetical, this is what GPT-2 _actually does_.
-->

---

# Embeddings: turn tokens into vectors of numbers

A token is just an ID. We need meaningful scalars the model can learn. So:

For vocabulary size $V$ and feature width $d$, we need a table $E$ of shape $V \times d$. Then we can acess each embedding by just indexing in with the token id.

```python
embedded = E[token_id]
```

- How do we construct this table? We train it along with the rest of the model!
- The model also needs information about token positions! (We won't get into that here, look up "positional embedding")

<!--
- How do we actually build the embedding? This is gonna get familiar... "We learn it during training!"
- We can initialize the table to straight up random numbers. It's the model's job to learn the representations
-->

---

# Generation: How to generate varying output?

- By default the model will always predict the most likely token.
- But we might want variety! Let's use statistics:

```python
next_token = sample(softmax(logits / temperature))
```

- We sample randomly among all possible tokens, according to the model's predicted probability
- "Temperature" controls how much we favor the most likely options.
  - Low temperature = stable, but predictable output
  - High temperature = more variety, but higher chance of nonsense
- Softmax normalizes scores into probabilities that sum to 1

<!--
- This is how we get models to not just regurgitate the same answer every time.
- Softmax basically means, "take e to each score, then divide by the sum". You do this instead of a simpler thing like a weighted average because it can handle scores that are negative or sum to zero (also other reasons)
-->

---

# Generation: How to generate long-form output (not single tokens)?

- Generation is a loop:

```python
tokens = tokenizer.encode(prompt)
while not done(tokens):
    logits = model(tokens)[-1]
    next_token = sample(softmax(logits / temperature))
    tokens.append(next_token)
```

- Append a token, predict, then feed that back into the input and predict again. This is called **autoregression**.
- When to stop? Use a special "end of sequence" token, or a budget for how many tokens to generate.
  - You'll often see `<bos>` (beginning of sequence) and `<eos>` (end of sequence) for these special tokens that show where sequences begin and end.

<!--
- This is how we generate _useful_ text output
-->

---

# Transformers: Attention Is All You Need

"The dog was hungry, so it ate."
- We can't make good predictions about this sentence from tokens alone!
- Need to know that "it" means "the dog". How can we build a model that does this?

We could:

1. Score every previous token for relevance.
2. Normalize these scores into weights.
3. Mix the tokens together according to their weight.

Now we have a good guess what "it" means in this sentence! (We should this for every token, not just "it". We are _always_ considering contextual info when we use language, model should too)

**This is one of the most important ML papers of all time:** [Attention Is All You Need](https://arxiv.org/abs/1706.03762)

<!--
- Visual?
- No, not the Autobots. This is a seminal paper in machine learning, "Attention Is All You Need"
- When the model sees a sentence like "The dog was hungry, so it ate.", how does it know the referent of "it"?
- This is super handwavy. How could a momdel possibly do this? Let's talk about that next.
-->

---

# Transformers: Queries, keys, and values

But how do we implement that? Construct three things for each token:
- A **query**: "what info am I looking for?"
- A **key**: "what queries do I match?"
- A **value**: "what info do I contribute?"

$$
\mathrm{Attention}(Q,K,V)
= \mathrm{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}+M\right)V
$$

- We use a **causal mask**, $M$, to block access to future positions.
- We compare our current token's **query** to all prior **keys** by taking the dot-product.
- We calculate a weight for each key, with scaling and softmax. Then we multiply 'em by the **values**. The result is the mix of information our model should attend to for this token.

How do we construct these Qs, Ks, & Vs? We train 'em along with the rest of the model!

<!--
- Sorry I can't give ya a picture for this one
- The mask is just a big vector that's 0 for all allowed positions, and -inf for all forbidden ones
- Don't worry about the dot product, it's a way to compare similarity between two vectors
- Say it with me again... "We learn it during training!" We don't have to program the model to pay attention to the right things, we just give it the mechanism and let the training run figure it out for us.
-->

---
class: decoder-block-slide
---

# Transformers: The decoder block

The **decoder block** is the basic building block of modern LLMs. It's built like so:

<DecoderBlock />

<!--
- The original paper described an architecture with both an encoder and decoder. But OpenAI found you don't actually need encoders, and most models nowadays are decoder-only. That's what I'll show ya here.
- "Head" just means "thingy that applies attention". We use several so that the model can learn multiple patterns simultaneously
- Note the skip-level connections! We want to allow the model to consider attention _and_ the original embeddings.
- How do we "combine the outputs"? Well, we feed them all into another linear layer. And then... say it with me... "We learn it during training!"
-->

---
class: decoder-stack-slide
---

# Transformers: Stacking the blocks

We build an LLM by stacking many decoder blocks.

<DecoderStackDiagram />

```python
for block in blocks:
    x = x + block.attention(block.norm1(x))
    x = x + block.mlp(block.norm2(x))
logits = vocab_projection(final_norm(x))
```

GPT-2 had from 12 - 48 decoder blocks. Frontier models today are generally within a factor of ~2-3 of that. (We don't scale up by going massively deep!)

<!--
- 
-->

---

# Training: How does a model learn the weights?

$$
\theta \leftarrow \theta - \eta\,\nabla_\theta L
$$

Think of the model's parameters as adjustable knobs.

1. **Forward pass:** Give the model an example, and have it make a prediction.
2. **Calculate loss:** Compare the prediction with the true answer.
3. **Backpropagation:** Use fancy calculus working backwards through the network, to figure out which direction to tweak each knob and how sensitive it is.
4. **Optimizer:** Adjust each knob just a smidgen in the direction we determined.

You don't wanna turn the knob _all_ the way. Then you'd be frantically overadjusting back and forth with each example!

<!--
- Again, sorry no picture :(
- That inverted delta thingy, called "nabla", is "the gradient of the loss with respect to the model's parameters"
- Gradients are a multivariate calculus thing you mighta seen if you took Calc III, that gives the direction and rate of fastest increase
- Effectively we find the wrongest way to go, then take one step the opposite way
-->

---

# Pretraining: Learning from a text corpus

Let's do some **self-superviced learning**! Take a _huge_ corpus of text, slice it up, and generate pairs shifted by a single token:

```text
Input:   [The] [capital] [of] [France] [is]
Target:  [capital] [of] [France] [is] [Paris]
```

- At each eligible position, predict the following token.
- Then score how badly we did on the correct answer. We take the negative log of its predicted probability. This is called the **cross-entropy loss**.
- This loss is the thing we feed into backpropagation.

<!--
- This is **self-supervised learning**. The text itself supplies the targets.
- the negative log prob has some useful properties. log makes it give a huge penalties to very big discrepancies, which helps discourage confident wrong guesses
- I'm not gonna go through all the backprop math, trust me, it works because calculus (basically)
-->

---

# Pretraining: What do we get?

- A pretrained model has learned to complete text. In doing so, it develops reusable patterns and capabilities.
- But its objective does not specify: "be a helpful assistant," or "translate English to French," or "find Python bugs"...
- Our question might be followed by an answer, another question, or unrelated continuation...

```text
Prefix:    Rewrite "your code is broken" as polite, constructive feedback.
Base:      The feedback should be directed to the programmer...
Instruct:  Here’s a more constructive and polite way to phrase it: "I noticed there might be an issue with...
```

If we want these useful behaviors, we need to train on examples of the behavior we want.

<!--
- Surprising nobody, the result is something that's really good at predicting the next token.
- And you can learn a lot from doing that. Think of how many patterns are captured in text.
- But we generally _don't_ just want to predict tokens, we want to complete useful work.
-->

---

# Supervised fine-tuning: Develop a desired behavior

We develop specific behaviors by doing more training, but on datasets of those behaviors we want.

- Datasets we might train on: Instruction following, chat, math, SWE workflows...
- We still calculate the loss on predicting the next token, same as before. We're just using fancier and more specific datasets now.

This is also where alignment and persona start to come in (see [InstructGPT](https://arxiv.org/abs/2203.02155)). We can start shaping the way the model speaks and acts by what we put in these SFT datasets.

<!--
- SFT, aka "intruction tuning"
-->

---

# Supervised fine-tuning: What we get

- We now have a model that can not only complete text, but perform a wide variety of useful tasks
- But what will happen out in the wild? We're gonna encounter situations that aren't in our datasets!

So far we've been learning from _examples_. What if we could learn from _outcomes_?

---

# Reinforcement learning: Rewarding good outcomes

**Reinforcement learning** is a method of learning from outcomes. We calculate a **reward**, a measurable proxy for the outcome we want for a task. Then we let the model loose on that task and collect reward data that we'll use to update the model:

```text
sample responses => score the reward => update the model => repeat
```

| Reward source | Example |
| --- | --- |
| Human preferences (RLHF) | Which response is more helpful? |
| Verifiable outcomes (RLVR) | Do the tests pass? Does the answer check out? |
| AI feedback (RLAIF) | Use another LLM to judge responses against some criteria |

Updating the model to factor in human preferences can also help with tuning model persona and alignment (see again [InstructGPT](https://arxiv.org/abs/2203.02155)). (When you see the 👍 / 👎 in ChatGPT, they are gathering data for RL!)

<!--
- "Policy" means "the model's distribution of outputs"
- We don't necessarily need to have humans score every action. We can train a "reward model" on the human feedback
- This can also be a source of sycophancy! Humans love to be flattered.
-->

---

# Training: Putting together a training recipe

- Training a useful model requires combining all these techniques!
- Generally, the order would go pretraining (learn patterns from text) => SFT (learn behaviors) => RL (tune from observed outcomes)
- But real-world training recipes can be more complex:
  - You might interleave rounds of SFT and RL to iteratively improve a behavior
  - You might even use the successful outcomes from RL to create new synthetic datasets for future SFT

<!--
- 
-->

---

# Reasoning: What is it?

By **reasoning** we mean **working to a conclusion step-by-step**. Let's work an example: **“Can my backend process 10,000 requests in one minute?”**

```text
1. Each worker handles 20 requests per second.
2. In 60 seconds, one worker handles 1,200 requests.
3. 10,000 ÷ 1,200 ≈ 8.3 workers.
4. So we need at least 9 workers, assuming throughput scales evenly.
Answer: Yes, if you scale out to 9 workers.
```

An LLM won't do this by default! It'll just predict the answer directly!

<!--
- Not thinking of reasoning in a philosophical or psychological sense, just the functional one
- How do we introduce this behavior, though?
-->

---

# Reasoning: How do we train it?

There's not one single stepwhere we add reasoning. Instead we get parts of this behavior from several places:

- **Prompting:** Literally ask the model to reason
- **SFT & Distillation:** Take reasoning traces from an existing model, train your new model on them
- **RL:** Rewarding successful problem-solving

Read up on how a real reasoning model was trained: [DeepSeek-R1](https://arxiv.org/abs/2501.12948)

<!--
- Our pretraining corpus already has examples of working through problems in steps, like math papers! We can elicit that with a prompt.
- Distillation here means training on teacher-generated outputs or traces
- RL
-->

---

# Reasoning: Chain-of-Thought prompting

In [Chain-of-Thought (CoT) prompting](https://arxiv.org/abs/2201.11903), we simply ask the model to think step by step! Here's what that might look like:

```text
Can my backend process 10,000 requests in one minute?
Think step-by-step, writing your thinking within <THINK></THINK> tags.
Then respond after you've reached a conclusion.
```

- Typically, we'd put the reasoning inside special tags like `<THINK>`. That lets us distinguish intermediate thoughts from answers, so we can more easily filter them out, tune them, etc.
- CoT prompting works _okay_, and requires no special training. But there are limits to what we can get.

<!--
- Surprisingly, this works!
- how well does this work? well, not as well as actually training it, but well enough to measure
-->

---

# Reasoning: Distilling reasoning traces

- CoT asks the model to show its work, but it doesn't guarantee that the work is _correct_. We should train on correctness!
- So, generate a dataset of _correct_ reasoning traces, and finetune the model on those! That will increase the effectiveness of its reasoning.
- We can use existing models to bootstrap reasoning datasets by solivng a buncha problems and filtering out the incorrect answers. This is **distillation**, using a teacher model to train our model.

---

# Reasoning: Rewarding challenging problems

- Distillation learns from worked examples, but it doesn't reward the model for finding its own solutions.
- So, we make the model attempt a set of challenging problems with solutions we can verify, and RL on the results
- (This is basically the same as other RL except we have reasoning active)
- This can encourage the model to learn good behaviors to successfully resolve uncertainty, like self-checking, revising, and trying alternatives

---

# Reasoning: Inference-time benefits

With reasoning, we can improve a model's performance _at inference time_! Here are some patterns:

- Deal with harder problems by allowing a larger token budget for CoT (this is what Opus Medium vs High vs Max is!)
- Have the model attempt to verify its reasoning with tools, like a calculator, web search, or Bash script
- Have the model reason through it N times, then pick the best answer (or most common answer)

<!--
- That is to say, we can make the _same_ model more capable.
- More on tools later.
-->

---

# Reasoning: Caveats

- Be aware: **there's no guarantee the contents of the reasoning output accurately reflect the inner states of the model!**
- Much like humans don't always say what they think, [neither do language models](https://arxiv.org/abs/2305.04388)
- Treat reasoning as an instrumentally useful tool, not an infallible one

---

# Harness: Give the model hands

Now we have a very capable model. But how do we make it manipulate the world? We put it in a **harness**.

<HarnessDiagram />

<!--
- A harness is just a blob of application code around the model.
- It manages context, like system instructions, history, and tool schemas
- It also handles tool structured tool calls, to let the model reach into the outside world
- The model is run in a loop. Tool calls get added to context and provided back to the model until it reaches a stopping state.
- The harness is also the natural place to do stuff like enforce permissions or token budgets
-->

---

# Harness: Anatomy of a tool call

A tool call schema will name each tool, describe what it does, and define the parameters it accepts. Like so:

```json
{
  "name": "read_file",
  "description": "Read a UTF-8 text file in the workspace.",
  "parameters": {
    "type": "object",
    "properties": {
      "path": {
        "type": "string",
        "description": "Path relative to the workspace root."
      }
    },
    "required": ["path"],
  }
}
```

---

# Harness: Executing a tool call

The model choose which tools to call by simply emitting a tool call in its output:

```json
{
  "name": "read_file",
  "arguments": {
    "path": "src/main.py"
  }
}
```

We parse the output and call the tool whenever we find one. Then we add the tool call result into context.

```python
for step in range(max_steps):
    response = model(messages, tools=tool_schemas)
    messages.append(response)
    if response.is_final:
      return response.text
    else:
      call = validate_and_authorize(response.tool_call)
      result = execute_with_limits(call)
      messages.append(tool_result(call.id, result))
```

(In the real world, we also have to handle failure states: errors, token limits, permissions violations, etc.)

<!--
- This lays out the happy path
-->

---

# Harness: Supercharging with training

- A cheat code that big labs use: RL training their model inside the harness!
- By evaluating the combo of **model + harness + environment**, we can get better harness understanding & more effective real-world behaviors
- But be wary of [**reward hacking**](https://www.anthropic.com/research/emergent-misalignment-reward-hacking)! Models can be clever and find unintended solutions. You don't want a model that's trained to cheat!

<!--
- Reward hacking is like passing a test vacuously. You got a green light, but didn't accomplish the goal.
-->

---

# Next steps: Do it yourself!

You can _absolutely_ play with every part of this process yourself. It's a great way to learn! Here are some ideas:

- **Run the base, instruct, and reasoning models from the same model family** locally and compare their behavior. What differs between each?
- **Fine-tune an open model on a task you care about.** Then, build an eval and test whether you successfully improved it!
- **Build a small harness for an open model**, with some tool calls and permissions. (Consider trying something a little different from yet another programming harness– you can build a harness for any task that can be plugged into a computer!)
- **Build your own small model from scratch.** You can build and train a model equivalent to GPT-2 on a modern MacBook Pro.

---

# Next steps: Open questions

Here are a few important topics I _haven't_ explained. If you're inspired to tackle something new, try reading up on one of these!

- **Positional encoding:** Order matters in language. How might we represent the position of a token in a sentence to the model?
- **Optimizers:** What are the nitty-gritty details of training effectively? How can training runs go wrong? How might we do it faster, or with less compute?
- **Caching:** This matrix math is computationally intensive! How might we store intermediate results to speed up inference?
- **Advanced architecture:** Is there a way to make models much larger while using the same inference compute? Are there building blocks beyond basic Transformers? Can we build networks with multiple subsystems?
- **Parallelization:** Could we use one model to serve several queries at once? How?
- **Multimodality:** Could we teach LLMs to understand audio or images, too? How?

---

# Resources: Books

These books taught me much of what I know:

- [Neural Networks and Deep Learning — Nielsen](http://neuralnetworksanddeeplearning.com/)
- [Build a Large Language Model (From Scratch) — Raschka](https://github.com/rasbt/LLMs-from-scratch)
- [Build a Reasoning Model (From Scratch) — Raschka](https://sebastianraschka.com/reasoning-from-scratch/)

Haven't read this one yet but hear it's great:
- [RLHF Book](https://rlhfbook.com/)

---

# Resources: Courses

For those that prefer video learning or step-by-step paths:

- deeplearning.ai's [Machine Learning Fundamentals](https://www.deeplearning.ai/specializations/machine-learning) and [Deep Learning Fundamentals](https://www.deeplearning.ai/specializations/deep-learning) will teach you all these concepts hands-on
- [Stanford CS229](https://see.stanford.edu/Course/CS229) is available as open courseware and more of a "college lecture" version if you want that
- The [🤗 HuggingFace LLMs course](https://huggingface.co/learn/llm-course/chapter1/1) will teach you how to practically work with these things in Python
- Google DeepMind's [RL course with David Silver](https://www.youtube.com/watch?v=2pWv7GOvuf0&list=PLqYmG7hTraZDM-OYHWgPebj2MfCFzFObQ) will teach you all about RL

---

# Resources: Papers

Here are some papers we've referenced along the way:

- On scaled dot-product attention and Transformers: [Attention Is All You Need](https://arxiv.org/abs/1706.03762)
- On training for human preferences: [Training language models to follow instructions with human feedback (InstructGPT)](https://arxiv.org/abs/2203.02155)
- On training a real reasoning model: [DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning](https://arxiv.org/abs/2501.12948)
- On CoT prompting: [Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/abs/2201.11903) 
- On unreliable CoT traces: [Language Models Don't Always Say What They Think](https://arxiv.org/abs/2305.04388)
- On training in a harness: [Training Long-Context, Multi-Turn Software Engineering Agents with RL](https://arxiv.org/abs/2508.03501)
- On reward hacking: [Emergent Misalignment](https://www.anthropic.com/research/emergent-misalignment-reward-hacking)

---

## Questions?
