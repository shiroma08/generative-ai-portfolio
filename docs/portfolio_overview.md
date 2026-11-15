# Portfolio Overview

All four tasks use one running case: **TerraFlask**, a fictional reusable insulated water bottle (90% recycled stainless steel, sage-green finish, bamboo lid, 24-hour cold retention) sold to university students in hot-climate cities. Keeping one theme means each task's findings feed the next, and the Week 5 artefact is evaluated against evidence gathered across the portfolio. The case links to **SDG 12 (Responsible Consumption and Production)**, through reusable products and honest environmental claims, and **SDG 10 (Reduced Inequalities)**, through accessible alt text and fair service across languages.

## Task 1: Model comparison
**Notebook:** [`week1-model-comparison/Week1_Model_Comparison.ipynb`](../week1-model-comparison/Week1_Model_Comparison.ipynb)

One product brief was given to an open-weight language model (Qwen2.5-3B-Instruct, three runs at temperature 0.3 and 0.9) and an open-weight image model (Stable Diffusion v1.5, six runs varying guidance, steps and seed). Both outputs were scored against the same seven-point checklist and the lab's 1–5 rubric.

- **Language model:** met all applicable checklist items (17/20 on the rubric). It still added small claims the brief never made, such as "durable" and "comfortable grip", and used the vague term "eco-friendly".
- **Image model:** produced attractive photorealistic images (10/20) but rendered the bamboo lid in none of six images, could not write the tagline, and placed a hot-climate product in a temperate European park in every image.
- **Why:** a language model can copy facts from its prompt; a diffusion model's CLIP text encoder registers which concepts are present but not which object each attribute belongs to.

## Task 2: Prompt engineering
**Notebook:** [`week2-prompt-engineering/Week2_Prompt_Engineering.ipynb`](../week2-prompt-engineering/Week2_Prompt_Engineering.ipynb)

Task: extract environmental claims from product listings into JSON and rate how well they are substantiated, a first-pass greenwashing screen. A baseline prompt, an engineered prompt (role, schema, rules, label definitions, one worked example) and an ablation without the example were each run on eight listings, three sampled runs plus one greedy run each (96 generations), and scored against an answer key.

- **Format:** the schema and JSON-only instruction took valid output from 0% to 100%.
- **Classification:** the worked example raised rating accuracy from 0.38 to 0.79 and halved hallucinated values.
- **Remaining failures:** all came from judgement, not extraction. Examples: measurable claims treated as vague; "ships in a cardboard box" flagged as a claim; two percentages summed into an invented 90%.

## Task 3: RAG and fine-tuning
**Notebooks:** [`Week3A_RAG_Pipeline.ipynb`](../week3-rag-and-finetuning/Week3A_RAG_Pipeline.ipynb), [`Week3B_QLoRA_Finetuning.ipynb`](../week3-rag-and-finetuning/Week3B_QLoRA_Finetuning.ipynb), [`Week3_Technical_Report.ipynb`](../week3-rag-and-finetuning/Week3_Technical_Report.ipynb)

- **RAG:** eight company documents, chunked, embedded with MiniLM, indexed in FAISS and answered by Qwen2.5-3B with a grounding prompt. Twelve test questions covered factual, multi-chunk, out-of-scope and adversarial cases, scored on context precision and recall, judge-based faithfulness and human review. The pipeline answered 8 of 12 correctly and refused both out-of-scope questions. It also repeated a planted prompt-injection claim, invented a price, and refused two answerable questions. A chunking ablation compared 200, 500 and 1,000-character chunks.
- **Fine-tuning:** QLoRA (4-bit NF4, r = 16) on 800 Dolly closed-QA examples adapted Qwen2.5-0.5B in 10.4 minutes. It raised ROUGE-L from 0.15 to 0.34, human-rated correctness from 3.0 to 3.7 out of 5, and cut answers with invented details from 7 to 2, at the cost of over-short answers.

## Task 4: Image generation and captioning
**Notebook:** [`week4-image-generation-captioning/Week4_Image_Generation_and_Captioning.ipynb`](../week4-image-generation-captioning/Week4_Image_Generation_and_Captioning.ipynb)

The notebook tries to fix the Week 1 image failures. It uses a one-change-at-a-time prompt ladder, five seeds, four schedulers and ControlNet Canny conditioning from a drawn silhouette. BLIP-2 then captioned and answered questions about every image, and CLIP scored prompt and caption alignment.

- **Setting:** naming a desert setting fixed the setting failure. **Lid:** the word "bamboo" landed on the background, the body or the whole bottle, but almost never the lid; only ControlNet produced the specified lid reliably.
- **Captioning:** BLIP-2 captions were accurate for 17 of 21 images. Question answering accepted false premises ("bamboo" for the lid of an image with no bottle).
- **Evaluation:** CLIP-based checks shared Stable Diffusion's own blind spot and ranked the most faithful images lowest, so every automated score needed human review.

## How the models work

**Transformer language models (Qwen2.5).**
- Text is split into tokens, and each token becomes a vector.
- In each layer, **self-attention** lets every token weigh every earlier token, so the model can refer back to facts given anywhere in the prompt.
- The model generates one token at a time from a probability distribution over the vocabulary. **Temperature** rescales that distribution, and **top-p** keeps only the most likely tokens.
- **Instruction tuning** on prompt–response pairs and preference feedback turns a base model that continues text into one that follows requests.
- Because training rewards *likely* text rather than *verified* text, models can produce fluent, plausible statements that are false. That is hallucination, which RAG reduces by putting verified text into the prompt.

**Latent diffusion models (Stable Diffusion).**
- A CLIP text encoder turns the prompt (at most 77 tokens) into embeddings.
- A U-Net starts from random noise in a compressed latent space (64 × 64 × 4) and removes noise step by step, steered by the text through cross-attention. The **scheduler** sets the step sizes and the **seed** sets the starting noise.
- A VAE decoder turns the final latent into a 512 × 512 image.
- **Classifier-free guidance** compares the U-Net's prediction with and without the prompt and pushes the image further towards the prompt.
- **ControlNet** adds a trainable copy of the U-Net's encoder that takes a second input, such as an edge map, so the composition follows a given structure.

**Generative adversarial networks (GANs).** Before diffusion, image generation was dominated by GANs, in which a generator and a discriminator are trained against each other. GANs sample in a single pass and produce sharp images, but adversarial training is unstable and prone to **mode collapse**, where the generator covers only part of the data. Diffusion models train with a simple, stable denoising objective, cover the data distribution more fully and accept text conditioning naturally. That is why they replaced GANs for text-to-image generation, at the cost of slower, multi-step sampling.

**Retrieval-augmented generation and parameter-efficient fine-tuning.** RAG retrieves relevant passages with embedding similarity and places them in the prompt, so answers can be grounded in and traced to sources without retraining. **LoRA** freezes the pre-trained weights and learns small low-rank update matrices in selected layers. **QLoRA** stores the frozen base model in 4-bit precision, so a model can be adapted on a single consumer GPU. RAG changes what a model *knows* at answer time; fine-tuning changes how it *behaves*.

## References
- Dettmers, T. et al. (2023) 'QLoRA: efficient finetuning of quantized LLMs', *Advances in Neural Information Processing Systems*, 36.
- European Parliament and Council (2016) *Regulation (EU) 2016/679 (General Data Protection Regulation)*. Official Journal of the European Union, L 119.
- European Parliament and Council (2024) *Regulation (EU) 2024/1689 (Artificial Intelligence Act)*. Official Journal of the European Union, L 2024/1689.
- Goodfellow, I. et al. (2014) 'Generative adversarial nets', *Advances in Neural Information Processing Systems*, 27.
- Ho, J., Jain, A. and Abbeel, P. (2020) 'Denoising diffusion probabilistic models', *Advances in Neural Information Processing Systems*, 33.
- Ho, J. and Salimans, T. (2022) 'Classifier-free diffusion guidance', arXiv:2207.12598.
- Holtzman, A. et al. (2020) 'The curious case of neural text degeneration', *International Conference on Learning Representations*.
- Hu, E.J. et al. (2022) 'LoRA: low-rank adaptation of large language models', *International Conference on Learning Representations*.
- Lewis, P. et al. (2020) 'Retrieval-augmented generation for knowledge-intensive NLP tasks', *Advances in Neural Information Processing Systems*, 33.
- Li, J. et al. (2023) 'BLIP-2: bootstrapping language-image pre-training with frozen image encoders and large language models', *International Conference on Machine Learning*.
- Mitchell, M. et al. (2019) 'Model cards for model reporting', *Proceedings of the Conference on Fairness, Accountability, and Transparency*, pp. 220–229.
- Qwen Team (2024) 'Qwen2.5 technical report', arXiv:2412.15115.
- Radford, A. et al. (2021) 'Learning transferable visual models from natural language supervision', *International Conference on Machine Learning*.
- Reimers, N. and Gurevych, I. (2019) 'Sentence-BERT: sentence embeddings using Siamese BERT-networks', *Proceedings of EMNLP-IJCNLP*.
- Rombach, R. et al. (2022) 'High-resolution image synthesis with latent diffusion models', *IEEE/CVF Conference on Computer Vision and Pattern Recognition*.
- Vaswani, A. et al. (2017) 'Attention is all you need', *Advances in Neural Information Processing Systems*, 30.
- Zhang, L., Rao, A. and Agrawala, M. (2023) 'Adding conditional control to text-to-image diffusion models', *IEEE/CVF International Conference on Computer Vision*.
