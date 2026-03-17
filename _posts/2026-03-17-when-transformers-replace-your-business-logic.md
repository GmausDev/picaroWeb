---
title: "When Transformers Replace Your Business Logic"
date: 2026-03-17 10:00:00 +0100
layout: post
---

**X's January 2026 decision to replace its entire Scala-based recommendation engine with a single Grok transformer marks the most visible example of a deeper industry shift: neural networks are systematically eating hand-coded software.** This migration—from hundreds of hand-engineered features and rule-based heuristics to a unified transformer that learns relevance from raw behavioral sequences—crystallizes what Andrej Karpathy predicted in 2017 as "Software 2.0." The implications ripple far beyond recommendation systems. Tesla deleted 300,000 lines of C++ planning code in favor of end-to-end neural networks. Google replaced hand-tuned search ranking with deep learning models. Netflix now treats user history as token sequences fed to foundation models. Traditional software architecture principles like DDD, Hexagonal Architecture, and Clean Architecture aren't disappearing—but their domain is shrinking as learned systems consume territory that once belonged to explicitly programmed logic.

---

## X's algorithm: from Scala spaghetti to a single transformer

### The old system (open-sourced March 2023)

Twitter's original recommendation pipeline, published at `github.com/twitter/the-algorithm`, was a **multi-stage Scala/JVM architecture** orchestrated by "Home Mixer," built atop Product Mixer—a custom Scala framework for composing content feeds. The pipeline ran approximately **5 billion times daily**, completing in under 1.5 seconds but consuming 220 seconds of CPU time per execution.

The process worked in stages. First, candidate sourcing retrieved roughly 1,500 tweets from hundreds of millions—split approximately 50/50 between in-network posts (ranked by Earlybird, a Lucene-based search index) and out-of-network discovery (powered by SimClusters for community embeddings, TwHIN for knowledge graph embeddings, and UTEG for user-tweet entity graphs). A light ranker—a logistic regression model trained years earlier—narrowed candidates to about 400. Then the heavy ranker, a **~48-million-parameter Parallel MaskNet neural network**, predicted engagement probabilities across ten-plus actions. Finally, extensive hand-coded Scala heuristics applied filtering and re-ranking.

The scoring weights revealed the system's priorities: `P(reply_engaged_by_author)` carried a weight of **75.0** (the strongest positive signal), while `P(report)` carried **-369.0**. Critically, the model used no content-based embeddings—all features were hand-engineered engagement and social signals. Maintaining this system required multiple teams managing separate models, feature pipelines, and thousands of lines of heuristic business logic.

### The new system (open-sourced January 20, 2026)

X's rebuilt algorithm, published at `github.com/xai-org/x-algorithm`, represents a fundamental architectural break. The orchestration layer was rewritten entirely in **Rust** (replacing Scala), while the ML core moved to **Python with JAX/Haiku** (replacing PyTorch/TensorFlow). The centerpiece is Phoenix—a **Grok-based transformer** ported from xAI's Grok-1 architecture, adapted for recommendation.

![X's Architecture Migration: Before vs After](/assets/images/x-architecture-migration.svg)

The repository contains four main components. **Thunder** is a Rust-based in-memory post store with Kafka ingestion that provides sub-millisecond lookups for in-network content. **Home-mixer** is the Rust gRPC orchestration server. **Candidate-pipeline** provides a generic Rust trait-based pipeline framework. **Phoenix** houses the ML models: a two-tower retrieval model (encoding users and posts into vectors for dot-product similarity matching) and the Grok-based ranking transformer that processes engagement history sequences.

The README makes the design philosophy explicit: *"We have eliminated every single hand-engineered feature and most heuristics from the system."* The transformer directly processes behavioral sequences—what you liked, replied to, shared—to predict relevance across **15 engagement actions** (11 positive, 4 negative). A critical architectural choice called "candidate isolation" ensures that during transformer inference, candidates cannot attend to each other, only to user context. This makes scores consistent and cacheable.

The four-stage scoring pipeline (Phoenix Scorer → Weighted Scorer → Author Diversity Scorer → Out-of-Network Scorer) evaluates thousands of posts in **under 200 milliseconds**, a significant improvement from the old system's latency profile. The migration eliminated the light ranker entirely, collapsed multiple embedding systems into hash-based lookups, and replaced the fragmented multi-model architecture with a single unified transformer.

### Why X made the switch

Three forces drove the migration. First, **reduced complexity**: the old system's 30+ separate models, hand-engineered features, and Scala heuristics created enormous maintenance overhead. A single transformer handling ranking eliminates most of that coordination cost. Second, **adaptability through end-to-end learning**: instead of teams manually engineering features and tuning weights, the transformer autonomously learns user-content relevance from behavioral sequences—improvements come from better data and model capacity, not debugging business rules. Third, **regulatory pressure**: the EU fined X **€120 million** in December 2025 under the Digital Services Act for insufficient algorithmic transparency, and Paris prosecutors opened an investigation into suspected algorithmic bias in July 2025. Open-sourcing the new system addressed these pressures directly.

Notable caveats: the release was a single atomic commit with no development history, contained no model weights or training data, and the Rust code shipped without Cargo.toml build files—making independent reproduction impossible.

---

## Karpathy's three paradigms and the death of hand-coded logic

Andrej Karpathy's November 2017 essay "Software 2.0" laid the intellectual foundation for understanding this shift. His core argument: neural networks are not merely another tool in the ML toolbox—they represent **a fundamentally new way of writing software**. In Software 1.0, programmers write explicit instructions. In Software 2.0, they define architectures and objectives, and optimization algorithms discover the program (as neural network weights) through gradient descent. The programmer's role shifts from specifying behavior to curating data and designing loss functions.

![Karpathy's Three Software Paradigms](/assets/images/software-paradigms.svg)

Karpathy's evidence was already compelling in 2017: visual recognition had moved from engineered features plus SVMs to ConvNets; speech recognition from Gaussian mixture models and Hidden Markov Models to end-to-end neural networks; machine translation from phrase-based statistical methods to transformers. He quoted Fred Jelinek's 1985 observation—*"Every time I fire a linguist, the performance of our speech recognition system goes up"*—as prophecy fulfilled.

At his June 2025 Y Combinator keynote, Karpathy formalized a three-layer framework. **Software 1.0** is traditional programming. **Software 2.0** is neural network weights discovered through optimization. **Software 3.0** is LLMs programmed via natural language prompts—the unit of programming becomes the prompt, not the function. He drew directly from his Tesla experience: *"There was a ton of C++ code around in the autopilot...over time, the neural network grew in capability and size, and all the C++ code was being deleted."*

His November 2025 update introduced a crucial framing for understanding where this paradigm applies: *"Software 1.0 easily automates what you can specify. Software 2.0 easily automates what you can verify."* Tasks with resettable environments, efficient iteration, and automated reward signals—math, code, games—progress rapidly under Software 2.0. Creative and strategic tasks, harder to verify, lag behind. This explains the "jagged frontier" of AI capability.

Yann LeCun independently reframed this shift as **"differentiable programming"** in January 2018, emphasizing that modern software is increasingly built by assembling parameterized functional blocks trained through gradient-based optimization. The philosophical implications are stark: the transition moves from specification to optimization, from determinism to probabilism, and from interpretability to raw capability.

---

## The industry follows: Tesla, Google, Netflix, and beyond

X's migration is not an isolated event. Across the technology industry, learned systems are systematically replacing hand-coded logic in domains where complexity exceeds human ability to specify rules.

![Industry Adoption: Hand-coded Logic Replaced by Learned Systems](/assets/images/industry-adoption.svg)

**Tesla's Full Self-Driving v12** represents the most dramatic example. In late 2023, Tesla replaced over **300,000 lines of C++ planning and control code** with a single end-to-end neural network trained on millions of video clips. The remaining code shrank to roughly 2,000-3,000 lines needed to activate and manage the neural networks. As Tesla's Dhaval Shroff explained: *"Instead of determining the proper path of the car based on rules, we determine the car's proper path by using neural networks that learn from millions of training examples of what humans have done."* The improvement trajectory changed fundamentally—instead of debugging intricate C++ conditionals, engineers curate training data and scale model capacity.

**Google Search** underwent a parallel transformation. Before RankBrain launched in 2015, search ranking was coded entirely by hand—teams of engineers analyzing queries, running tests, and implementing changes to hundreds of individually weighted ranking factors. RankBrain was the first deep learning system in search, and Google has since added RankEmbed and DeepRank. The transition enabled daily model rollouts where weekly manual updates once struggled to keep pace.

**Netflix** evolved from Cinematch (basic collaborative filtering circa 2000) through the Netflix Prize era's matrix factorization to today's system combining deep neural networks, reinforcement learning, contextual bandits, and transformer-based foundation models. Their latest approach treats user interaction history as token sequences—functionally identical to how LLMs process language—building a unified foundation model. Over **80% of what people watch** comes from algorithmic recommendations.

**YouTube's** 2016 paper documented the transition to a two-stage deep learning architecture with approximately one billion parameters trained on hundreds of billions of examples. **Spotify** moved from human-curated playlists to ML-powered personalization using collaborative filtering, NLP on web text, and convolutional neural networks on raw audio spectrograms. **Amazon** progressed from its pioneering item-to-item collaborative filtering (2003) to deep autoencoders and neural collaborative filtering, with a "once-in-a-decade leap" via deep learning autoencoders in 2019.

The pattern is consistent: **hand-engineered features and rule-based heuristics yield to end-to-end learned systems** wherever sufficient training data exists and the task is complex enough that explicit rules become unmaintainable.

---

## Where DDD, hexagonal, and clean architecture still matter

A 2024 IEEE/ACM study found that ML pipelines show *"limited use of OO mechanisms and minimal adherence to software design principles"* with *"poor code organization and deficient levels of coupling and cohesion."* This reflects a genuine tension—but not the death of traditional architecture.

Google's seminal 2015 paper "Hidden Technical Debt in Machine Learning Systems" provides the critical framing: **only a small fraction of real-world ML systems is actual ML code**. The surrounding infrastructure—data collection, feature computation, monitoring, serving, configuration—is vast and complex. Traditional architectural principles remain essential for this infrastructure. The paper identified ML-specific anti-patterns that directly violate classical principles: boundary erosion (breaking separation of concerns), entanglement where "Changing Anything Changes Everything" (violating single responsibility), glue code that freezes systems to specific ML packages, and pipeline jungles of organically evolved data preparation.

Clean Architecture's core insight—separating business logic from infrastructure concerns—remains **directly applicable to the infrastructure surrounding ML models**. Dependency Inversion enables switching between Azure OpenAI, Vertex AI, and AWS Bedrock without touching core logic. Hexagonal Architecture's ports-and-adapters pattern maps naturally to model serving interfaces. DDD's bounded contexts help partition systems where some domains suit ML and others require explicit rules.

But the ML model itself resists these patterns fundamentally. Models are not decomposable into single-responsibility modules. Their behavior emerges from data, not readable logic. Testing requires statistical validation rather than deterministic assertions. As Capital One's engineering team articulated: *"Rules are a good fit when exact logic is known and precision-based. Machine learning fits when exact logic is NOT known."* The practical reality is **hybrid architecture**: explicit rules for deterministic business logic and compliance constraints, ML for prediction, optimization, and pattern recognition in complex domains.

Mary Shaw of Carnegie Mellon, speaking at ICSE 2025, offered a measured perspective: *"Generative AI is another disruptive new idea. It is fundamentally statistical rather than algorithmic... However, SE has seen software with these properties before—not in quite the same form, but close enough to have robust techniques and theories."* Software engineering can evolve to accommodate these new subsystems. The discipline doesn't become irrelevant—it becomes more important for the **95%+ of system code surrounding the model**.

---

## The price of opacity: what transformer models cannot give you

The shift from explicit architectures to learned models comes with concrete, measurable costs that practitioners must weigh carefully.

**Debuggability degrades fundamentally.** In DDD or Hexagonal Architecture, every decision traces to a specific line of code. With a transformer, the logic is distributed across billions of parameters. Post-hoc explanation tools like SHAP and LIME provide approximations, not true explanations. O'Reilly's Patrick Hall notes that *"for many failure modes, you need to understand what the model is doing in order to understand what went wrong and how to fix it."* Worse, ML systems fail silently—they can run to completion without exceptions while producing incorrect results. A model can be confidently wrong with no signal that anything is broken.

**Testing loses its deterministic foundation.** Jeremy Jordan, a PyTorch Lightning core maintainer, observes that *"while traditional software tests have metrics such as the lines of code covered when running tests, this becomes harder to quantify when you shift your application logic from lines of code to parameters of a machine learning model."* The industry still lacks established ML test coverage conventions. Emerging approaches include behavioral testing (invariance tests, directional expectation tests, minimum functionality tests from the ACL 2020 CheckList paper) and continuous production monitoring for data drift and concept drift. But these are statistical approximations, not the deterministic guarantees that unit tests provide.

**Domain knowledge becomes implicit.** When business rules are encoded in code, the code itself documents domain logic—this is DDD's central insight. When those rules are absorbed into model weights, the accumulated understanding becomes opaque. Model cards and data sheets are emerging practices, but they cannot replicate the self-documenting nature of explicit bounded contexts and domain entities.

**Regulatory compliance grows harder.** GDPR Article 22 requires *"meaningful information about the logic involved"* in automated decisions. The EU AI Act (compliance required by August 2026) demands high-risk systems be *"designed and developed in such a way to ensure their operation is sufficiently transparent."* Cynthia Rudin's research at Duke argues that for many high-stakes decisions, **interpretable models can match black-box performance**, making the accuracy-interpretability trade-off a false dichotomy. Her work on the 2018 Explainable ML Challenge demonstrated that interpretable models achieved comparable accuracy on the same tasks.

**Production reliability shifts from deterministic to statistical.** Unlike code that behaves identically until changed, models degrade through data drift and concept drift. Unity Software's AI tool degradation cost an estimated **$110 million in sales** (~8% of revenue). COVID-19 diagnostic models were found relying on rulers visible in X-ray images rather than actual pathology. Amazon's hiring AI, trained on a decade of predominantly male resumes, systematically discriminated against women. IBM Watson for Oncology provided unsafe cancer treatment recommendations due to reliance on synthetic rather than real patient data. These failures share a common root: the opacity of learned systems hides failure modes that explicit logic would surface immediately.

The failure case inventory makes the argument for hybrid approaches compelling. Rule-based guardrails handle safety-critical logic, compliance requirements, and known business constraints. ML handles pattern recognition in high-dimensional spaces where rules cannot scale. Modern LLM deployments already exemplify this: Anthropic's Constitutional AI, OpenAI's RLHF, and Meta's Llama Guard all combine rule-based and neural approaches for content safety.

---

## Conclusion: the expanding stack, not the replacement stack

The narrative of AI "replacing" traditional software architecture misses the more nuanced reality. **What's happening is an expanding stack**, not a replacement. Contemporary AI-powered systems operate across all three of Karpathy's paradigms simultaneously: Software 1.0 handles API routing, authentication, database queries, and compliance logic; Software 2.0 powers classification, ranking, and prediction through trained models; Software 3.0 enables natural language interfaces and flexible reasoning through LLMs.

![The Expanding Stack: Where Each Paradigm Applies](/assets/images/hybrid-architecture.svg)

X's migration illuminates where the frontier is moving. The domains where learned systems outperform coded logic share common properties: **high-dimensional input spaces, complex non-linear relationships, sufficient training data, and tasks where "good enough" probabilistic answers beat brittle deterministic rules.** Recommendation, perception, natural language understanding, and planning all fit this profile.

But the domains where explicit architecture remains superior are equally clear: **regulatory compliance, safety-critical constraints, deterministic business rules, auditability requirements, and any context where "why did it decide this?" must have a concrete answer.** The most capable engineering organizations—and Google's technical debt research confirms this—are those that architect clear boundaries between these domains, applying traditional patterns rigorously to infrastructure and integration while letting learned systems handle the pattern recognition they do best.

The key insight for software engineers is not that their skills become obsolete, but that the **center of gravity shifts**. Writing business logic gives way to curating training data. Debugging code gives way to debugging data quality and model drift. Designing algorithms gives way to designing loss functions and evaluation frameworks. And perhaps most importantly, the architectural skill of knowing *which paradigm to apply where*—when to write an explicit rule versus when to train a model—becomes the defining capability of the next generation of system designers.
