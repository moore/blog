---
title: "Ai Is Not a Simulation"
date: 2023-11-22T08:04:35-08:00
draft: false
---

I was having a conversation with a group of AI researchers, and we eventually turned to the topic of whether AI is a simulation of human intelligence. I believe this view is incorrect and potentially dangerous.

I think it's more accurate to view our goal in AI as creating systems that are at least as *capable* as humans, though they will most likely operate very differently.

The closest model we have to the human mind is [Spiking Neural Networks](https://en.wikipedia.org/wiki/Spiking_neural_network). However, despite some advantages, they are unlikely to become the mainstream approach for practical AI applications. [Geoffrey Hinton](https://en.wikipedia.org/wiki/Geoffrey_Hinton) has identified a key limitation of the spiking model: it can only learn linearly, whereas other models, such as the widely-used Transformer, can learn in parallel. Multiple instances of the [Transformer](https://en.wikipedia.org/wiki/Transformer_(machine_learning_model)) model can learn from various examples and then be combined into a single model that encompasses the sum of what was learned individually.

There are other notable differences too:

- Each human brain has a unique arrangement of connections between neurons, which means one could never directly copy memories from one brain to another. In contrast, our digital AI models can be run on any universal computer.

- Backpropagation used in transformers (which cannot work in spiking models) is a more efficient way to learn, allowing for more extensive learning from each example shown.

- In terms of parameter counts, human brains are much larger than our largest models. However, our large models have learned from larger datasets than any human ever could.

But why is it wrong to think of AI models as simulations of human intelligence?

If we regard AI as similar to us, this could lead to [consensus bias](https://en.wikipedia.org/wiki/False_consensus_effect), where we assume that AI systems think about situations in the same way we do. This assumption will likely lead us to mispredict their behavior, potentially in dangerous ways.

Furthermore, by thinking of AIs as simulations of us, I believe we inadvertently strip them of agency. While considering AI as entities with rights and goals is controversial today and may not currently be practical, we must take this idea seriously as advancements continue. I hope that when we eventually discuss AI agency, we see AIs as having their own unique identities, not as a lesser form of ourselves.

