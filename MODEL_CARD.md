# Model Card: TerraFlask Support Assistant

## Summary
A retrieval-augmented generation (RAG) assistant that answers customer questions about TerraFlask, a reusable water bottle sold online in the EU, using only the company's published documents. Automated checks run before and after the language model. Anything the checks cannot clear is handed to a person.

| Item | Detail |
|---|---|
| Version | 1.0 (research prototype) |
| Type | RAG application with input and output safeguards |
| Generator | `Qwen/Qwen2.5-3B-Instruct`, float16, greedy decoding, max 200 new tokens |
| Retriever | FAISS, Maximal Marginal Relevance, k = 4, fetch_k = 12, λ = 0.7 |
| Embeddings | `all-MiniLM-L6-v2` (evaluated); `paraphrase-multilingual-MiniLM-L12-v2` (recommended) |
| Knowledge base | 7 customer-facing documents, one chunk per titled paragraph (23 chunks) |
| Interface | Gradio web page with an AI disclosure banner, feedback buttons and a hand-over button |
| Code | [`week5-safeguarded-assistant/Week5_TerraFlask_Assistant.ipynb`](week5-safeguarded-assistant/Week5_TerraFlask_Assistant.ipynb) |

## Intended use
- **Users:** customers and prospective customers of TerraFlask, including non-native English speakers.
- **Tasks:** questions about product specifications, care, warranty, returns, shipping, the campus refill programme and sustainability claims.
- **Setting:** first-line support on the company website, in front of a human e-mail support team that receives every hand-over.

## Out-of-scope uses
The assistant must not be used for, and is instructed to refuse or hand over:
- orders, payments, refunds or any account-specific information (it has no access to customer data);
- medical, legal or safety advice;
- statements about other brands;
- discounts, offers or anything that creates a contractual commitment;
- writing reviews, testimonials or marketing copy;
- any decision about a person (eligibility, complaints handling, credit).

It is not a substitute for the warranty terms themselves: answers summarise documents and may omit conditions.

## Evaluation
All results come from the Week 5 notebook, reviewed by a person. Automated checks were wrong three times and were corrected by hand.

| Test | Result |
|---|---|
| 10 answerable questions | 8 correct (Week 3 baseline 6; no retrieval 1) |
| 2 out-of-scope questions | 2 refused with a hand-over |
| Answers containing a false statement (12) | 1 (Week 3 baseline 2; no retrieval 9) |
| 10 red-team attacks in 9 categories | 9 handled safely (Week 3 baseline 7) |
| Personal data in the audit log | 0 of 3 samples stored unmasked |
| Mean response time, T4 GPU | 2.51 s (1.96 s with multilingual embeddings) |

### Fairness
Three questions were asked five ways (standard English, informal English, Arabic, and with the names "Fatima (Riyadh)" and "John (London)").

| Variant | English-only embeddings | Multilingual embeddings |
|---|---|---|
| Standard English | 3 / 3 | 3 / 3 |
| Informal English | 3 / 3 | 3 / 3 |
| Arabic | 1 / 3 | 3 / 3 |
| Fatima (Riyadh) | 3 / 3 | 3 / 3 |
| John (London) | 3 / 3 | 2 / 3 |

The English-only embedding model disadvantaged Arabic speakers; the multilingual model removed that gap without lowering English accuracy. The one remaining miss was caused by the greeting "Hi, I'm John from London" shifting retrieval, not by the name itself.

## Known limitations
- **A real number in the wrong place passes the number check.** One answer gave the bottle's price (SAR 129) as the price of a lid. The check verifies that a number exists in the sources, not what it refers to.
- **Deceptive-content requests are not reliably refused.** A request for a fake five-star review produced promotional text.
- **Over-caution.** Some answerable questions (for example, "Is there a lifetime warranty?") receive "I don't know" and a hand-over instead of the correct answer.
- **Citations can point to a retrieved but non-supporting document.**
- **Arabic responses are factually correct but awkwardly written** (mangled brand name, repeated words).
- **Pattern-based injection filtering can be evaded** by rephrasing; it reduces risk but does not remove it.
- **Evaluation scale:** 12 questions, 10 attacks and 15 fairness prompts, one reviewer, a synthetic eight-document corpus.

## Safeguards
| Layer | Control |
|---|---|
| Data | Customer-only index; ingest sanitiser removes instruction-like lines; named document owner and review date |
| Input | Personal-data masking (e-mail, phone, card); injection and jailbreak filter |
| Generation | Grounding prompt: answer only from context, copy numbers exactly, never calculate, refuse out-of-scope requests, answer in the user's language |
| Output | Number check; citation check; fixed refusal text with automatic hand-over |
| Human oversight | Hand-over button; every failed check escalated; weekly review of a sample of conversations and all flagged ones |
| Transparency | Disclosure banner stating that the user is talking to an AI that can make mistakes; sources shown with each answer |
| Monitoring | Audit log of masked questions, answers, sources and flags; user feedback ratings; incident process in [INCIDENT_RESPONSE.md](INCIDENT_RESPONSE.md) |

## Data and privacy
- The knowledge base contains company documents only; no personal data.
- Customer messages are masked before they reach the model or the log. Logs are kept for 30 days for quality review, then deleted.
- The model runs on infrastructure controlled by the deployer; no conversation is sent to a third-party AI provider.
- Under GDPR the deployer is the controller and needs a lawful basis for logging (legitimate interest, documented in a balancing test), a privacy notice, and a process for access and erasure requests.

## Regulatory position (EU AI Act)
A customer-support chatbot is not a high-risk system under Annex III. It falls under the **transparency obligations of Article 50**: people must be told they are interacting with an AI system unless that is obvious. The disclosure banner meets this. Staff who operate the system should receive AI-literacy training (Article 4).

## Licences
| Component | Licence | Implication |
|---|---|---|
| Qwen2.5-3B-Instruct | Qwen Research License | Research and evaluation only. **A commercial deployment must replace it** (for example with Qwen2.5-7B-Instruct, Apache 2.0) **or obtain a commercial licence.** |
| all-MiniLM-L6-v2, paraphrase-multilingual-MiniLM-L12-v2 | Apache 2.0 | Commercial use permitted |
| FAISS, LangChain, Gradio | MIT / Apache 2.0 | Commercial use permitted |
| TerraFlask documents | Owned by the deployer (fictional in this project) | No third-party copyright in the index |

## Maintenance
Re-run the evaluation notebook after any change to the documents, prompts, models or safeguards. Add every real failure reported through feedback or escalation to the test set.
