# Formal Mathematical Representation Discovery

## A Research Thesis for AI-Native Mathematics

### 1. Central Thesis

Modern formal mathematics libraries such as Lean/mathlib [mathlib Community, 2020] are not merely repositories of theorems and proofs. They are large-scale, machine-readable records of how mathematical knowledge is represented, compressed, transported, and reused. Their internal structures—definitions, typeclasses, proof states, tactic traces, dependency graphs, coercion paths, simplification rules, and theorem statements—encode a computational shadow of mathematical abstraction itself.

The central thesis of this research program is:

> The next major advance in AI for mathematics will not come only from better next-tactic prediction, larger language models, or brute-force proof search. It will come from learning how mathematical representations are discovered, reused, compressed, and transferred across domains.

A formal mathematics system such as Lean gives us, for the first time, an empirical substrate for studying this process. It exposes the topology of proof search, the dependency geometry of mathematical concepts, the recurring motifs of proof construction, and the places where existing representations are insufficient. By mining these structures, we can build systems that retrieve analogies, suggest intermediate lemmas, detect missing abstractions, and eventually help invent new mathematical representations.

This research direction can be summarized as **formal mathematical representation discovery**.

Its goal is not simply to prove more Lean theorems. Its goal is to understand and automate the higher-level acts that make proof possible: choosing the right object, introducing the right invariant, transporting structure across domains, and compressing repeated proof patterns into reusable abstractions.

---

### 2. Motivation

Current AI theorem proving is largely framed as a local search problem. A model sees a proof state and predicts a tactic, a lemma, or a proof term. Recent systems such as AlphaProof [Google DeepMind, 2025], DeepSeek-Prover-V2 [DeepSeek-AI, 2025], and HyperTree Proof Search [Lample et al., 2022] have demonstrated that this framing, combined with large-scale data and reinforcement learning, can achieve remarkable results. Yet the framing is incomplete.

Human mathematical creativity rarely consists of local tactic selection alone. Much of it consists of representation change.

A difficult theorem often becomes tractable only after one introduces a new object, defines a better interface, proves an intermediate lemma, recognizes an analogy with another field, or reformulates the problem in a more natural language. In mature mathematics, these representational moves are often more important than the final proof script.

Formalized mathematics contains abundant evidence of this phenomenon. In mathlib, many definitions and structures exist not merely because they are mathematically natural in isolation, but because they make large families of theorems reusable. A typeclass hierarchy, a bundled structure, a coercion mechanism, a normal form, or a carefully shaped lemma can dramatically reduce proof complexity across hundreds of downstream results.

This suggests a new view:

> A mathematical abstraction is valuable when it compresses proof search and increases theorem transportability.

Lean makes this view measurable. We can ask:

- Which structures reduce proof length and dependency depth?
- Which lemmas act as bottlenecks or bridges between domains?
- Which proof-state motifs recur across apparently unrelated areas?
- Which theorem statements have analogous relational structure despite different terminology?
- Where do proofs repeatedly suffer from the same representational friction?
- Can we detect missing abstractions before humans define them?

These questions move beyond theorem proving as local search and toward theorem proving as representation engineering.

---

### 3. Related Work

#### 3.1 AI Theorem Proving Systems

The dominant paradigm in AI theorem proving frames proof search as a sequential decision problem. AlphaProof [Google DeepMind, 2025] combines a language model with AlphaZero-style reinforcement learning inside Lean, achieving silver-medal performance at the 2024 IMO. DeepSeek-Prover-V2 [DeepSeek-AI, 2025] uses RL-guided subgoal decomposition to achieve 88.9% pass rate on MiniF2F. HyperTree Proof Search [Lample et al., 2022] applies an online AlphaZero variant over and-or proof trees. These systems advance the state of the art in automated proof but operate entirely within the tactic-prediction paradigm this thesis moves beyond.

#### 3.2 Premise Selection and Retrieval-Augmented Proving

Premise selection—choosing which library lemmas to supply to a prover—is the earliest form of library-aware proof automation. The hammer tradition [Paulson and Blanchette, 2010; Blanchette et al., 2016] introduced tools like Sledgehammer for Isabelle, with premise filters based on symbol overlap [Meng and Paulson, 2009] and machine-learned classifiers [Kühlwein et al., 2013]. Kaliszyk and Urban [2014] demonstrated that ML-guided premise selection at scale substantially increases automated proof coverage. LeanDojo [Yang et al., 2023] extends this to Lean 4, providing programmatic proof-state extraction and a retrieval-augmented prover (ReProver) that selects premises using program-analysis-guided contrastive training. LeanSearch [Gao et al., 2024] provides semantic search over Mathlib4 for informal natural-language queries. COPRA [Thakur et al., 2023] uses GPT-4 with retrieved lemmas and stateful backtracking.

This thesis's analogy retrieval component (§5.3) is related to premise selection but differs in aim: rather than retrieving lemmas to supply to a prover, it retrieves structurally analogous theorems as a source of proof strategy, using multi-layer graph structure rather than text or symbol overlap.

#### 3.3 Graph-Based Representations of Formal Libraries

Two bodies of work have applied graph analysis to formal proof libraries, forming the closest technical predecessors to this thesis.

Paliwal et al. [2020] and Bansal et al. [2019] (HOList) apply graph neural networks to HOL Light formula graphs for tactic and premise prediction. These systems use graph structure within individual theorem statements but do not mine cross-theorem motifs or study abstraction at the library level.

Li et al. [2026] present a network analysis of Lean 4's Mathlib itself, extracting its dependency structure into a multilayer graph with over 308,000 declarations and millions of edges across thousands of modules. They introduce graph decompositions distinguishing explicit edges from compiler-synthesized ones and study macroscopic structural properties. This is the closest existing work to the graph extraction proposed in §5.1. The key difference is scope and use: Li et al. study Mathlib's network structure descriptively to characterize the library; this thesis proposes to use graph structure operationally for motif retrieval, analogy search, and abstraction detection.

Huch [2022] models the Isabelle Archive of Formal Proofs as a complex network (1.8M nodes, 2.8M edges) and studies centrality metrics, finding small-world structure and scale-free in-degree distributions. Like Li et al., this work examines static dependency centrality and does not pursue motif mining, analogy retrieval, or abstraction detection.

Blanchette et al. [2015] mine the AFP empirically for proof statistics: proof size, tactic usage distributions, dependency structure, and proof style evolution. This is the most direct precursor to the motif mining proposal in §5.2. The key distinction is that Blanchette et al. report descriptive statistics over tactic sequences; this thesis proposes to extract structured motifs from proof-state transition graphs and use them operationally for retrieval and abstraction suggestion.

PACT [Han et al., 2021] extracts self-supervised training tasks from Lean proof terms. LeanDojo [Yang et al., 2023] extracts proof states, premises, and tactic traces from Lean 4 at scale. The trace collection layer (§6 Layer 1) builds on and extends this infrastructure.

#### 3.4 Proof-Pattern Mining and Tactic Discovery

**Proof-pattern mining.** The most direct conceptual ancestors of this thesis are the ML4PG family of systems for Coq/SSReflect. ML4PG [Komendantskaya et al., 2012] is a machine learning plugin that clusters proof scripts by statistical features extracted from proof states and tactic sequences, identifying similarity across proof developments. Heras and Komendantskaya [2014] demonstrate that ML4PG can mine electronic Coq proof libraries and suggest proof strategies by analogy in "Recycling Proof Patterns." This is the clearest ancestor of Hypothesis 1 and §5.2 (Proof Motif Mining). The key distinctions from this thesis are: ML4PG uses shallow statistical features over tactic sequences rather than structured proof-state transition graphs; it operates on Coq/SSReflect at a scale far smaller than Lean/mathlib; and it does not address theorem analogy retrieval or missing abstraction detection.

Freitas and Whiteside [2014] identify named, reusable "proof patterns" for formal verification—recurring solutions to common proof obligations—through manual expert identification; this thesis proposes automated discovery from data at scale.

**Tactic discovery.** TacMiner [Xin et al., 2025] is the closest prior work to Project 1 (§7). It builds Tactic Dependence Graphs (TDGs) over Lean 4 proofs to identify reusable proof strategies across multiple proofs, discovers new tactic library entries, and refactors proofs into more modular forms—reporting 3× as many learned tactics as Peano and a 26% proof-size reduction. TacMiner is the state of the art in automated tactic discovery from proof graphs. The distinction from this thesis is one of scope: TacMiner targets tactic library compression as an end in itself; this thesis uses motif structure as a stepping stone toward theorem analogy retrieval, missing abstraction detection, and representation discovery at library scale.

#### 3.5 Mathematical Analogy in Automated Reasoning

Work on proof analogy in automated reasoning dates to the 1980s. Boy de la Tour and Caferra [1987] propose second-order pattern matching to transform known proof terms into candidates for analogous propositions. Melis and Veloso [1994] use derivational analogy at the level of proof plans to guide automated provers by replaying known analogous proofs. This thesis pursues analogy at a different level of abstraction—structural graph similarity across a large formal library—and aims at retrieval rather than direct proof transfer.

The cognitive science foundation for relational analogy is structure-mapping theory [Gentner, 1983; Falkenhainer et al., 1989], which holds that analogical reasoning aligns relational structure between domains rather than surface features. The Structure-Mapping Engine (SME) operationalizes this as an algorithm over relational descriptions. The multi-layer graph retrieval proposed in §5.3 can be viewed as a large-scale, data-driven realization of structure-mapping principles in a formal mathematical setting.

#### 3.6 Theory Exploration and Conjecture Generation

Theory exploration systems automatically discover lemmas in a given formal theory development. Hipster [Johansson et al., 2014] integrates theory exploration into Isabelle/HOL using QuickSpec-style random testing to generate equational conjectures, then attempts to prove or prune them. LeanConjecturer [Onda et al., 2025] generates university-level mathematical conjectures for Lean 4 using LLMs, producing 12,289 conjectures from 40 Mathlib seed files.

Both are conceptually related to §5.4 (Missing Abstraction Detection) in that both attempt to identify what mathematical statements are "missing" from a library. This thesis's approach differs by using proof-state transition graphs and dependency topology to detect where the library lacks abstractions—rather than generating conjectures by random testing or language model sampling—and by targeting structural pain points (repeated motifs, long coercion chains, high-branching bottlenecks) that indicate missing lemmas, typeclasses, or normal forms.

#### 3.7 Library Learning and Abstraction Discovery

The program synthesis community has studied abstraction discovery as a library learning problem. DreamCoder [Ellis et al., 2021] jointly learns reusable library abstractions and a neural search policy through wake-sleep iteration: during the wake phase it synthesizes programs; during the sleep phase it refactors common subprograms into reusable library components using a compression-based criterion. Stitch [Bowers et al., 2023] provides a significantly more efficient top-down abstraction algorithm for the same setting, using program compression ratio as the primary metric of abstraction quality. LAPS [Wong et al., 2021] extends DreamCoder with natural language annotations to guide library learning.

These systems are the program synthesis analogues of what this thesis proposes for formal mathematics. The key differences are: (i) the objects are types and propositions rather than input-output examples; (ii) correctness is guaranteed by Lean's type checker rather than test suites; (iii) the abstraction space includes typeclasses, coercions, and universal properties, which have no direct program synthesis analogue; and (iv) the corpus is a mature human-curated library rather than a set of synthetic tasks.

#### 3.8 Perspectives on Mathematical Abstraction

Mitchell [2021] reviews AI approaches to abstraction and analogy-making, concluding that no current system achieves human-level analogical generalization and proposing evaluation criteria. Zhang et al. [2023] examine AI for mathematics from a cognitive science perspective, arguing that current systems lack the representation-change capabilities that make human mathematical reasoning powerful—a diagnosis this thesis takes as its starting point.

---

### 4. Core Hypotheses

This research program rests on five concrete hypotheses.

#### Hypothesis 1: Proof traces contain recurring cross-domain motifs.

Lean proofs are not arbitrary tactic sequences. They contain recurring structural patterns: introducing witnesses, decomposing conjunctions, applying extensionality, transporting along equivalences, normalizing coercions, invoking universal properties, reducing goals by induction, or closing branches through simplification.

These motifs often recur across different mathematical domains. The surface syntax may differ, but the proof-state topology may be similar. Prior empirical work on the Isabelle AFP [Blanchette et al., 2015] and manual pattern identification [Freitas and Whiteside, 2014] support the existence of such patterns; this thesis proposes to discover them automatically from proof-state transition graphs in Lean/mathlib.

If this hypothesis holds, proof traces can be mined for reusable proof motifs that are more abstract than tactic sequences and more operational than informal proof strategies.

#### Hypothesis 2: Theorem analogy can be retrieved through relational structure, not just text.

Many important mathematical analogies are not lexical. Two theorems may share little vocabulary but have similar dependency neighborhoods, type structures, proof skeletons, or relational patterns.

A theorem retrieval system based only on names, comments, or embeddings of surface syntax will miss many such analogies. Structure-mapping theory [Gentner, 1983; Falkenhainer et al., 1989] predicts that the relevant similarity is relational, not featural. A retrieval system based on multi-layer formal graphs may find relational analogies that text-based systems such as LeanSearch [Gao et al., 2024] miss.

The relevant object is not a theorem as text, but a theorem as a structured neighborhood in a formal mathematical graph.

#### Hypothesis 3: Missing abstractions leave measurable traces.

When a library lacks the right abstraction, users compensate through repeated proof patterns, duplicated lemmas, long coercion chains, brittle rewrite sequences, and similar theorem statements spread across namespaces.

These phenomena should be detectable. A system can identify clusters of repeated representational pain and suggest that a new lemma, typeclass, structure, normal form, or interface may be needed. This turns abstraction discovery into a mathematical refactoring problem analogous to code clone detection [Roy and Cordy, 2007; Koschke, 2007], applied to formal proof rather than source code.

#### Hypothesis 4: Abstraction quality can be measured by compression.

A good abstraction reduces the cost of future proof. It compresses repeated reasoning, shortens proofs, increases reuse, decreases search entropy, and creates stable interfaces between domains.

This suggests quantitative measures of abstraction value:

- proof length reduction;
- dependency reuse;
- decrease in branching factor;
- reduction of repeated proof motifs;
- increased theorem transportability;
- decrease in coercion and normalization burden;
- centrality of the abstraction in downstream proofs.

This compression-as-quality view is operationalized in program synthesis by Stitch [Bowers et al., 2023], which uses program compression ratio as the primary abstraction quality metric. A formal library allows us to study abstraction as compression in a richer and more rigorously typed setting.

#### Hypothesis 5: The topology of proof states predicts proof difficulty and useful intermediate lemmas.

A proof can be viewed as a trajectory through a space of proof states. Some regions are flat and locally searchable. Others have high branching, many irrelevant hypotheses, or require a long-range conceptual jump. Some intermediate states act as bottlenecks: once reached, many downstream goals become easy.

These bottleneck states may correspond to useful intermediate lemmas, invariants, or representation changes.

If this hypothesis holds, proof-state geometry can guide lemma invention and proof planning.

---

### 5. Research Agenda

The agenda has four mutually reinforcing components.

#### 5.1 Lean Graph Extraction and Normalization

The first task is to construct a graph-based representation of Lean/mathlib that is rich enough to capture mathematical structure but normalized enough to avoid irrelevant syntactic noise.

Existing tools provide a starting point. LeanDojo [Yang et al., 2023] extracts proof states, premise annotations, and tactic traces from Lean 4 repositories at scale. PACT [Han et al., 2021] demonstrates that rich training data can be extracted from Lean proof terms. This project extends such infrastructure to produce a multi-layer graph intermediate representation suitable for motif mining, analogy retrieval, and abstraction detection.

The system should extract multiple graph layers:

##### Declaration dependency graph

Nodes are definitions, theorems, instances, structures, and lemmas. Edges represent dependency, invocation, unfolding, instance use, or theorem application.

##### Expression graph

The theorem statement, definitions, and proof terms are represented as typed expression graphs. These capture binders, applications, constants, equalities, propositions, functions, inductive types, and typeclass constraints.

##### Typeclass and coercion graph

This graph records how mathematical structures are connected through instances, coercions, inheritance, bundled structures, and canonical constructions.

##### Proof-state transition graph

For tactic proofs, each tactic transforms one or more goals into successor goals. This induces a transition graph over proof states. The graph can record local context, target type, generated subgoals, tactic class, and closure mechanism.

##### Simplification and rewriting graph

`simp`, `rw`, normalization, and definitional equality hide substantial reasoning. The system should expose, where possible, the rewrite rules and normalization paths used to close goals.

The central design problem is normalization. Raw Lean expressions contain variable names, elaboration artifacts, syntactic choices, universe levels, implicit arguments, coercions, and proof-irrelevant details. The system must quotient out accidental differences while preserving mathematically meaningful structure.

A useful intermediate representation should support:

- alpha-renaming and binder normalization;
- namespace-aware abstraction;
- theorem statement canonicalization;
- typeclass neighborhood extraction;
- proof-state anonymization;
- tactic categorization;
- dependency slicing;
- graph embeddings and exact subgraph queries.

This infrastructure is the foundation for all later work.

---

#### 5.2 Proof Motif Mining

Once proof-state transition graphs are available, the next project is to discover recurring proof motifs.

A proof motif is a repeated local or global pattern in proof-state evolution. It is not merely a tactic sequence. It is a structural transformation pattern over goals, hypotheses, and generated subgoals.

Empirical analysis of the Isabelle AFP [Blanchette et al., 2015] and manually curated proof patterns for formal methods [Freitas and Whiteside, 2014] suggest that such recurring structures exist. The open question is whether they can be discovered automatically from proof-state transition graphs in a form useful for retrieval and proof planning.

Examples of expected motifs include:

- reduce equality of structured objects by extensionality;
- introduce an existential witness and split obligations;
- transform a goal through an equivalence and solve the transported version;
- prove a property by induction and close routine cases by simplification;
- use a universal property to convert a construction problem into a uniqueness problem;
- decompose algebraic structure into fields and discharge laws;
- normalize coercions until two expressions become definitionally equal.

The project would mine frequent subgraphs from proof-state transition graphs and cluster them by structural similarity. The result would be a library of proof motifs.

##### Evaluation

1. **Cross-domain recurrence**: Do motifs appear across multiple namespaces and mathematical areas?
2. **Predictive value**: Does motif identity help predict the next tactic or proof step?
3. **Retrieval value**: Given a proof state, can motif-aware retrieval find useful analogous proofs?
4. **Compression value**: Can motifs summarize many proofs more compactly than tactic sequences?
5. **Human interpretability**: Do mathematicians or Lean users recognize mined motifs as meaningful strategies?

The expected result is an empirical taxonomy of formal proof strategies, with quantitative evidence of cross-domain recurrence.

---

#### 5.3 Theorem Analogy Retrieval

The next component is a retrieval system that finds structurally analogous theorems across mathlib.

Given a theorem statement, proof state, or partial proof, the system should return other theorems that are similar not merely in text but in mathematical structure—motivated by structure-mapping theory [Gentner, 1983; Falkenhainer et al., 1989] and prior work on proof-plan analogy [Melis and Veloso, 1994].

Similarity can be computed using a combination of:

- theorem statement expression graphs;
- dependency neighborhoods;
- typeclass neighborhoods;
- proof motif signatures;
- proof-state transition summaries;
- namespace distance;
- shared abstraction usage;
- graph embeddings;
- exact or approximate subgraph matching.

The system should support queries such as:

- "Find theorems whose proof skeleton resembles this one."
- "Find results in other domains with a similar universal-property structure."
- "Find analogues of this order-theoretic theorem in topology, algebra, or category theory."
- "Find previous proofs where this kind of goal was solved after introducing an intermediate object."

This moves theorem retrieval from lexical search to structural analogy search.

##### Evaluation

1. **Known analogue pairs**: Human-curated pairs of analogous theorems as a held-out benchmark.
2. **Proof reuse prediction**: Whether retrieved theorems help prove held-out results.
3. **Namespace transfer**: Whether the system retrieves useful results from different mathematical domains.
4. **Ablation against baselines**: Compare (a) BM25 over theorem names and docstrings; (b) embedding-based search [Gao et al., 2024]; (c) symbol-overlap retrieval [Meng and Paulson, 2009]; (d) dependency-graph-only retrieval; (e) proof-motif-only retrieval; (f) full multi-layer graph retrieval.
5. **Expert judgment**: Ask Lean users whether retrieved analogies are mathematically meaningful.

The key claim to test is:

> Formal graph structure contains analogy information that is not captured by theorem names or text embeddings alone.

---

#### 5.4 Missing Abstraction Detection

The most ambitious component is a system that detects where mathlib lacks the right representation.

This turns abstraction discovery into a mathematical refactoring problem [Roy and Cordy, 2007]. Just as code clone detection identifies repeated code that should be factored into a shared abstraction, proof clone detection identifies repeated proof structure that signals a missing lemma, typeclass, or interface. The analogy with program synthesis library learning is also direct: both DreamCoder [Ellis et al., 2021] and Stitch [Bowers et al., 2023] identify reusable program substructures by finding patterns that improve compression; this thesis applies an analogous principle to formal proofs, exploiting the richer type-theoretic structure of Lean/mathlib.

The premise is that missing abstractions produce symptoms:

- repeated proof motifs across many files;
- families of similar theorem statements without a common interface;
- long chains of coercions or rewrites;
- frequent local definitions that resemble each other;
- clusters of lemmas that could be unified by a structure;
- high proof complexity around certain typeclass boundaries;
- proof scripts that perform the same construction repeatedly;
- theorem clusters with high internal similarity but low shared abstraction.

The system would identify such clusters and rank them by "representation inefficiency."

Possible outputs include:

- suggest a missing lemma;
- suggest a generalized theorem statement;
- suggest a new typeclass or structure;
- suggest a normal form;
- suggest a coercion or instance;
- suggest a library refactor;
- suggest an intermediate theorem that would reduce many proofs.

##### Evaluation

**Retrospective evaluation**: Use mathlib history. Identify moments when important abstractions were introduced. Ask whether the system, using only the pre-abstraction state of the library, would have detected the relevant cluster of representational pain.

**Prospective evaluation**: Run the system on current mathlib and produce suggestions. Have maintainers or domain experts judge whether the suggestions are meaningful. Measure whether accepted suggestions shorten proofs or simplify future development.

**Synthetic evaluation**: Remove or hide known lemmas and abstractions from a subset of the library and test whether the system rediscovers the need for them.

The central claim is:

> Missing mathematical representations leave detectable graph-theoretic and proof-theoretic traces before they are explicitly defined.

---

### 6. Technical Architecture

A plausible architecture has six layers.

#### Layer 1: Trace Collection

Instrument Lean to collect:

- theorem statements;
- proof terms;
- tactic traces;
- proof states before and after tactics;
- generated subgoals;
- used lemmas;
- typeclass resolution traces;
- simplification traces;
- coercion insertions;
- elaboration metadata.

The system should support both full-library batch extraction and interactive extraction inside an editor. Existing infrastructure such as LeanDojo [Yang et al., 2023] and PACT [Han et al., 2021] provides a foundation; the key extension is the proof-state transition graph and the normalization pipeline.

#### Layer 2: Formal Graph IR

Convert raw Lean data into a normalized graph intermediate representation.

This IR should be stable across superficial syntax changes and expressive enough to preserve mathematical relations. Work by Paliwal et al. [2020] and Bansal et al. [2019] on graph representations for HOL Light provides relevant prior art; the Lean 4 setting introduces additional structure through its typeclass and coercion system.

The IR should support:

- typed nodes and edges;
- multiple graph layers;
- slicing by theorem, namespace, dependency cone, or proof segment;
- canonical hashing of subgraphs;
- motif extraction;
- embedding generation;
- visualization.

#### Layer 3: Motif and Pattern Mining

Mine repeated structures:

- frequent proof-state transitions;
- theorem statement schemas;
- recurring dependency neighborhoods;
- repeated coercion/rewrite paths;
- common proof skeletons;
- bottleneck states.

This layer combines symbolic graph algorithms with learned embeddings. The static dependency analysis of Huch [2022] on the Isabelle AFP demonstrates that large formal library graphs have measurable large-scale structure; this layer extends that analysis to proof-state dynamics and uses the results operationally.

#### Layer 4: Retrieval and Analogy

Build retrieval models over the graph IR.

The primary approach is contrastive learning over multi-layer theorem graphs, with ablations against text-embedding baselines [Gao et al., 2024] and symbol-overlap baselines [Meng and Paulson, 2009]. Additional methods to explore include:

- graph kernels;
- subgraph matching;
- nearest-neighbor search over graph embeddings;
- hybrid symbolic-neural retrieval;
- LLM reranking with formal graph features.

#### Layer 5: Abstraction Scoring

Define metrics for representational inefficiency and abstraction value.

Examples:

- repeated motif density;
- proof compression ratio [Bowers et al., 2023];
- dependency centrality;
- namespace-spanning recurrence;
- typeclass boundary friction;
- search entropy;
- proof-state branching factor;
- normalized proof length;
- theorem transportability.

#### Layer 6: Interactive Assistant

Expose the system to users as an assistant for formal mathematics.

Given a proof state, it can say:

- "This resembles the proof pattern used in these theorems."
- "A useful intermediate lemma may be of this shape."
- "These hypotheses appear repeatedly in similar proofs."
- "This goal may be easier after transporting along this equivalence."
- "This family of proofs suggests a missing abstraction."

The final system is not merely an auto-prover. It is a representation-aware mathematical assistant.

---

### 7. Initial Projects

This research thesis should begin with concrete projects that are small enough to execute but strong enough to establish the direction.

#### Project 1: Proof-State Motif Mining in Mathlib

Build a Lean trace pipeline and mine recurring proof-state transition motifs across a subset of mathlib.

##### Research question

Do formal proofs contain recurring structural motifs that are stable across domains and predictive of proof strategy?

##### Deliverables

- proof-state graph extractor;
- motif mining algorithm;
- motif taxonomy;
- visualization tool;
- evaluation on tactic prediction and theorem retrieval.

##### Paper claim

Proof-state transition graphs contain reusable cross-domain motifs that are not captured by tactic sequences alone.

---

#### Project 2: Structural Theorem Analogy Retrieval

Build a theorem retrieval system that uses theorem statement graphs, dependency neighborhoods, and proof motif signatures.

##### Research question

Can formal graph structure retrieve useful mathematical analogies beyond lexical or embedding-based theorem search?

##### Deliverables

- multi-layer theorem graph representation;
- retrieval benchmark with human-curated analogue pairs;
- comparison against text-embedding baselines [Gao et al., 2024] and symbol-overlap baselines [Meng and Paulson, 2009];
- case studies of cross-domain analogies.

##### Paper claim

Multi-layer formal graph retrieval finds structurally analogous theorems across mathlib and improves proof reuse.

---

#### Project 3: Representation Inefficiency and Missing Lemma Discovery

Detect clusters of repeated proof motifs and propose intermediate lemmas or abstractions.

##### Research question

Can missing abstractions be detected through repeated proof patterns and graph-theoretic bottlenecks?

##### Deliverables

- representation inefficiency score;
- retrospective study using mathlib history;
- system-generated missing lemma candidates;
- evaluation through proof compression.

##### Paper claim

Formal libraries contain measurable representational pain points, and these can be used to suggest abstractions that reduce proof complexity.

---

### 8. Why This Is Different From Existing AI Theorem Proving

Most current systems attempt to automate proof search. They ask:

> Given a proof state, what tactic should be applied next?

This research asks a different question:

> Given a mathematical landscape, what representation would make proof search easier?

This difference matters.

Tactic prediction is local. Representation discovery is global.

Tactic prediction imitates proof scripts. Representation discovery studies why some proof scripts become short, reusable, and natural.

Tactic prediction operates within the current library. Representation discovery asks how the library itself should evolve.

Tactic prediction tries to close goals. Representation discovery tries to invent the objects that make goals close naturally.

This thesis also differs from the closest existing work on formal library analysis. Li et al. [2026] extract Mathlib's multilayer dependency graph and study its macroscopic structure, but examine only static dependencies and do not pursue proof-state dynamics, motif retrieval, or abstraction detection. Huch [2022] applies the same approach to the Isabelle AFP with the same limitation. Blanchette et al. [2015] mine the AFP for descriptive proof statistics but do not use the results operationally. TacMiner [Xin et al., 2025] builds Tactic Dependence Graphs over Lean 4 proofs to discover reusable tactic libraries—the closest prior work to Project 1—but targets tactic compression rather than representation discovery. All four works use formal library structure descriptively or for tactic automation; this thesis uses it as an active substrate for representation discovery.

The long-term goal is not just an AI that proves existing theorems, but an AI that helps mathematicians build better mathematical languages.

---

### 9. Why Lean Is the Right Substrate

Lean is especially suitable for this research because it combines several properties:

- a large and actively developed mathematical library [mathlib Community, 2020];
- expressive dependent type theory;
- rich typeclass mechanisms;
- tactic-based proof development;
- machine-checkable proof terms;
- substantial use of abstraction and hierarchy;
- enough library history to study abstraction evolution;
- an active community of expert formalizers;
- existing extraction infrastructure [Yang et al., 2023; Han et al., 2021].

The richness and complexity of the Lean typeclass system provides additional motivation. Baanen [2022] analyzes how mathlib uses Lean's instance parameter mechanism to organize its algebraic, order, topology, and analysis hierarchies, showing that representation choices in the typeclass system are non-trivial, consequential, and frequently interwoven in ways that create design tension. This supports the thesis's premise that Lean/mathlib contains substantial traces of representation decisions worth studying.

The HOL Light ecosystem provides a useful comparison point. HOList [Bansal et al., 2019] and the GNN work of Paliwal et al. [2020] demonstrate that graph-structured formal proof data supports meaningful machine learning. Lean/mathlib offers a richer typeclass hierarchy, a larger and more systematically organized library, and a more active community, making it a better substrate for studying abstraction at library scale.

Other proof assistants are also relevant. The Isabelle AFP has been analyzed graphically [Huch, 2022] and statistically [Blanchette et al., 2015], and the Mizar Mathematical Library has a long history as a formal corpus. These libraries may serve as secondary validation datasets to test whether mined motifs and detected abstractions generalize across systems.

Lean exposes both the high-level mathematical interface and the low-level proof machinery. This makes it possible to study the relationship between abstraction and proof search empirically.

---

### 10. Expected Contributions

This research program could contribute to multiple fields.

#### To AI theorem proving

It provides representation-aware retrieval, proof planning, intermediate lemma discovery, and abstraction-guided search.

#### To formal methods

It offers tools for library refactoring, proof maintenance, theorem organization, and abstraction quality measurement.

#### To programming languages

It treats formal mathematics libraries as evolving typed knowledge systems and studies abstraction as an empirical software phenomenon.

#### To machine learning

It provides new graph-structured learning problems grounded in rigorous symbolic data.

#### To mathematics

It may eventually help mathematicians discover analogies, transport ideas across fields, and identify the right objects to define.

#### To the philosophy and cognitive science of mathematics

It offers a computational way to study abstraction, analogy, representation, and proof compression—operationalizing theoretical frameworks such as structure-mapping theory [Gentner, 1983] on large-scale formal data [Mitchell, 2021; Zhang et al., 2023].

---

### 11. Long-Term Vision

The long-term vision is an AI-native environment for formal mathematics in which the system does not merely autocomplete proofs, but actively helps users navigate the space of mathematical representations.

A mathematician working in such an environment could ask:

- "What does this theorem resemble?"
- "Where has this proof pattern appeared before?"
- "What abstraction would unify these lemmas?"
- "What intermediate lemma would make this proof modular?"
- "Is this difficulty mathematical, or is it caused by library representation?"
- "Can this theorem be transported to another domain?"
- "What concept is missing from this part of the library?"

The system would answer using the formal structure of the library, not only natural language.

The program synthesis community has shown that library learning from a corpus of programs is feasible: DreamCoder [Ellis et al., 2021] and Stitch [Bowers et al., 2023] demonstrate that useful abstractions can be discovered by mining reusable structure and measuring compression. This thesis pursues an analogous program for formal mathematics, where the corpus is Lean/mathlib, the abstractions are mathematical definitions and typeclasses, and correctness is guaranteed by the type checker rather than by test suites. The richer type-theoretic structure of formal mathematics creates both additional challenges (the abstraction space includes coercions, universal properties, and bundled structures with no direct program synthesis analogue) and additional opportunities (graph structure is richer, and correctness is mechanically verified).

In this environment, Lean becomes more than a proof checker. It becomes a microscope for mathematical structure.

The most ambitious version of the thesis is:

> Formal mathematics libraries are the first large-scale datasets from which machines can learn the dynamics of mathematical abstraction.

If this is true, then the future of AI for mathematics is not merely automated proof. It is automated representation discovery.

The path from the three initial projects to this vision requires demonstrating, sequentially: (i) that proof-state motifs exist and are cross-domain; (ii) that graph-based analogy retrieval outperforms text-based retrieval on a held-out benchmark; and (iii) that the system's missing-abstraction suggestions correspond to genuine improvements in mathlib. Each step is independently publishable and provides the empirical foundation for the next.

---

### 12. Risks and Failure Modes

This direction has several risks.

#### Risk 1: Lean artifacts may overwhelm mathematical signal.

Proof states contain many artifacts from elaboration, typeclass resolution, coercions, naming choices, and library conventions. Graph mining may discover Lean-specific noise rather than mathematical structure.

**Mitigation**: Use normalization, quotienting, ablation studies, cross-namespace validation, human evaluation, and where possible comparison across different formalizations of the same mathematics.

#### Risk 2: Graph similarity may not correspond to meaningful analogy.

Two theorem graphs may look similar for superficial reasons, while deep analogies may be structurally distant.

**Mitigation**: Combine symbolic graph features with expert judgment, proof reuse evaluation, and retrieval-based downstream tasks. Benchmark against human-curated analogue pairs.

#### Risk 3: Missing abstraction detection may produce vague or unactionable suggestions.

It is easy to identify repeated patterns; it is harder to propose a clean abstraction.

**Mitigation**: Begin with missing lemma discovery and proof compression before attempting new structure invention. Measure whether suggestions reduce proof complexity.

#### Risk 4: Full proof-state extraction may be technically expensive.

Tracing all of mathlib may be large, brittle, or slow. LeanDojo [Yang et al., 2023] provides a starting infrastructure but is not optimized for full proof-state transition graph extraction.

**Mitigation**: Start with selected domains, cache aggressively, support incremental extraction, and focus first on tactic-level traces and dependency graphs.

#### Risk 5: The project may become too philosophical.

The big picture is attractive, but the field will only move if concrete systems and evaluations exist.

**Mitigation**: Anchor the research agenda in three initial artifacts: motif miner, analogy retriever, and missing abstraction detector. Each must have a concrete, reproducible evaluation.

---

### 13. Positioning Statement

This research direction should be positioned carefully.

It is not only "AI for theorem proving."

It is not only "graph mining on mathlib."

It is not only "retrieval for Lean."

It is the study and automation of how formal mathematical representations are created, compressed, reused, and transported.

A concise positioning statement is:

> We use Lean/mathlib as a machine-readable record of mathematical abstraction. By extracting and analyzing proof-state graphs, theorem dependency graphs, and typeclass structures, we build systems that discover proof motifs, retrieve cross-domain analogies, detect missing abstractions, and guide representation-aware proof search.

An even shorter version:

> This project turns formal mathematics libraries into datasets for learning mathematical representation discovery.

---

### 14. First Paper Sketch

A strong first paper should avoid overclaiming and establish one concrete pillar.

#### Possible title

**Proof Motifs in Formal Mathematics: Mining Reusable Proof-State Patterns from Lean/mathlib**

#### Core claim

Formal proof traces contain recurring structural motifs that appear across mathematical domains and improve theorem retrieval and proof-step prediction.

#### Contributions

1. A proof-state graph extraction pipeline for Lean 4, extending LeanDojo [Yang et al., 2023] with transition-graph structure.
2. A normalized representation of tactic-induced proof-state transitions.
3. A motif mining algorithm for formal proofs.
4. An empirical taxonomy of recurring proof motifs in mathlib.
5. Evidence that motif features improve theorem retrieval or proof guidance.

#### Evaluation

- motif recurrence across namespaces;
- prediction of next tactic category;
- retrieval of proof-similar theorems;
- human evaluation of motif interpretability by Lean users;
- ablation against (a) tactic-sequence baselines; (b) theorem-text baselines [Gao et al., 2024]; (c) dependency-graph-only baselines [Huch, 2022].

#### Why this paper matters

It demonstrates that formal proof traces contain reusable higher-level structure beyond tactic sequences. This justifies the broader agenda of representation-aware AI for mathematics and distinguishes this line of work from tactic prediction systems [Lample et al., 2022; Yang et al., 2023] and from prior static library analysis [Huch, 2022; Blanchette et al., 2015].

---

### 15. Final Thesis

The history of mathematics is partly the history of better representations. New objects, abstractions, and languages make previously impossible proofs natural. Until recently, this process was visible only through books, papers, lectures, and human interpretation. Formal mathematics changes that. It records mathematical representation in executable, machine-readable form.

Lean/mathlib [mathlib Community, 2020] therefore gives us a new scientific object: a large-scale formal trace of mathematical abstraction in action.

The opportunity is to build systems that can read this trace—not merely to imitate proof steps, but to learn the structural patterns by which mathematics organizes itself.

The research thesis is:

> By treating formal mathematics libraries as graph-structured records of representation, proof, and abstraction, we can build AI systems that retrieve deep analogies, discover reusable proof motifs, identify missing abstractions, and guide mathematicians toward better representations. This shifts AI for mathematics from local proof search toward representation discovery, the level at which much of mathematical creativity actually occurs.

This is the direction worth leading.

---

### References

Baanen, A. (2022). Use and abuse of instance parameters in the Lean mathematical library. In *Interactive Theorem Proving (ITP 2022)*, LIPIcs volume 237, pages 4:1–4:20. Schloss Dagstuhl. arXiv:2202.01629.

Bansal, K., Loos, S. M., Rabe, M. N., Szegedy, C., and Wilcox, S. (2019). HOList: An environment for machine learning of higher-order theorem proving. In *Proceedings of the 36th International Conference on Machine Learning (ICML)*, volume 97, pages 454–463.

Blanchette, J. C., Haslbeck, M., Matichuk, D., and Nipkow, T. (2015). Mining the archive of formal proofs. In *Intelligent Computer Mathematics (CICM 2015)*, LNCS volume 9150, pages 3–17. Springer.

Blanchette, J. C., Kaliszyk, C., Paulson, L. C., and Urban, J. (2016). Hammering towards QED. *Journal of Formalized Reasoning*, 9(1):101–148.

Bowers, M., Olausson, T. X., Wong, L., Grand, G., Tenenbaum, J. B., Ellis, K., and Solar-Lezama, A. (2023). Top-down synthesis for library learning. *Proceedings of the ACM on Programming Languages*, 7(POPL). arXiv:2211.16605.

Boy de la Tour, T. and Caferra, R. (1987). Proof analogy in interactive theorem proving: A method to express and use it via second-order pattern matching. In *Proceedings of AAAI-87*, pages 95–99.

DeepSeek-AI (2025). DeepSeek-Prover-V2: Advancing formal mathematical reasoning via reinforcement learning for subgoal decomposition. arXiv:2504.21801.

Ellis, K., Wong, C., Nye, M., Sablé-Meyer, M., Morales, L., Hewitt, L., Cary, L., Solar-Lezama, A., and Tenenbaum, J. B. (2021). DreamCoder: Bootstrapping inductive program synthesis with wake-sleep library learning. In *Proceedings of PLDI 2021*, pages 835–850. ACM. arXiv:2006.08381.

Falkenhainer, B., Forbus, K. D., and Gentner, D. (1989). The structure-mapping engine: Algorithm and examples. *Artificial Intelligence*, 41(1):1–63.

Freitas, L. and Whiteside, I. (2014). Proof patterns for formal methods. In *FM 2014: Formal Methods*, LNCS volume 8442, pages 279–294. Springer.

Gauthier, T., Kaliszyk, C., and Urban, J. (2017). Learning to reason with HOL4 tactics. In *21st International Conference on Logic for Programming, Artificial Intelligence and Reasoning (LPAR-21)*, EPiC Series in Computing, volume 46, pages 125–143. arXiv:1804.00595.

Gao, G., Ju, H., Jiang, J., Qin, Z., and Dong, B. (2024). A semantic search engine for Mathlib4. arXiv:2403.13310.

Gentner, D. (1983). Structure-mapping: A theoretical framework for analogy. *Cognitive Science*, 7(2):155–170.

Google DeepMind AlphaProof Team (2025). Olympiad-level formal mathematical reasoning with reinforcement learning. *Nature*. doi:10.1038/s41586-025-09833-y.

Han, J. M., Rabe, J., and van Doorn, F. (2021). Proof artifact co-training for theorem proving with language models. arXiv:2102.06203.

Heras, J. and Komendantskaya, E. (2014). Recycling proof patterns in Coq: Case studies. *Mathematics in Computer Science*, 8(1):99–116. arXiv:1301.6039.

Johansson, M., Rosén, D., Smallbone, N., and Claessen, K. (2014). Hipster: Integrating theory exploration in a proof assistant. In *Intelligent Computer Mathematics (CICM 2014)*, LNAI volume 8543, pages 108–122. Springer. arXiv:1405.3426.

Huch, F. (2022). Formal entity graphs as complex networks: Assessing centrality metrics of the archive of formal proofs. In *Intelligent Computer Mathematics (CICM 2022)*, LNCS volume 13467, pages 148–163. Springer.

Kaliszyk, C. and Urban, J. (2014). Learning-assisted automated reasoning with Flyspeck. *Journal of Automated Reasoning*, 53(2):173–213. arXiv:1211.7012.

Komendantskaya, E., Heras, J., and Grov, G. (2012). Machine learning in proof general: Interfacing interfaces. arXiv:1212.3618.

Koschke, R. (2007). Survey of research on software clones. In *Dagstuhl Seminar Proceedings 06301: Duplication, Redundancy, and Similarity in Software*.

Kühlwein, D., Blanchette, J. C., Kaliszyk, C., and Urban, J. (2013). MaSh: Machine learning for Sledgehammer. In *Interactive Theorem Proving (ITP 2013)*, LNCS volume 7998, pages 35–50. Springer.

Lample, G., Lachaux, M.-A., Lavril, T., Martinet, X., Hayat, A., Ebner, G., Rodriguez, A., and Lacroix, T. (2022). HyperTree proof search for neural theorem proving. In *Advances in Neural Information Processing Systems (NeurIPS)*, volume 35. arXiv:2205.11491.

Li, X., Peng, N., Severini, S., and Shafto, P. (2026). The network structure of Mathlib. arXiv:2604.24797.

mathlib Community (2020). The Lean mathematical library. In *Proceedings of the 9th ACM SIGPLAN International Conference on Certified Programs and Proofs (CPP)*, pages 367–381. arXiv:1910.09336.

Melis, E. and Veloso, M. (1994). Analogy makes proofs feasible. In *AAAI-94 Workshop on Case-Based Reasoning*.

Meng, J. and Paulson, L. C. (2009). Lightweight relevance filtering for machine-generated resolution problems. *Journal of Applied Logic*, 7(1):41–57.

Onda, N., Kasaura, K., Oriike, Y., Taniguchi, M., Sannai, A., and Sonoda, S. (2025). LeanConjecturer: Automatic generation of mathematical conjectures for theorem proving. arXiv:2506.22005.

Mitchell, M. (2021). Abstraction and analogy-making in artificial intelligence. *Annals of the New York Academy of Sciences*. arXiv:2102.10717.

Paliwal, A., Loos, S. M., Rabe, M. N., Bansal, K., and Szegedy, C. (2020). Graph representations for higher-order logic and theorem proving. In *Proceedings of the AAAI Conference on Artificial Intelligence*, volume 34, pages 2967–2974. arXiv:1905.10006.

Paulson, L. C. and Blanchette, J. C. (2010). Three years of experience with Sledgehammer, a practical link between automatic and interactive theorem provers. In *Proceedings of IWIL 2010*.

Roy, C. K. and Cordy, J. R. (2007). A survey on software clone detection research. Technical Report 541, Queen's University.

Thakur, A., Tsoukalas, G., Wen, Y., Xin, J., and Chaudhuri, S. (2023). An in-context learning agent for formal theorem-proving. arXiv:2310.04353.

Xin, Y., Xin, J., Poesia, G., Goodman, N., Chen, Q., and Dillig, I. (2025). Automated discovery of tactic libraries for interactive theorem proving. *Proceedings of the ACM on Programming Languages*, 9(OOPSLA2). arXiv:2503.24036.

Wong, C., Ellis, K., Tenenbaum, J. B., and Andreas, J. (2021). Leveraging language to learn program abstractions and search heuristics. In *Proceedings of the 38th International Conference on Machine Learning (ICML)*, volume 139, pages 11193–11204. arXiv:2106.11053.

Yang, K., Swope, A. M., Gu, A., Chalamala, R., Song, P., Yu, S., Godil, S., Prenger, R. J., and Anandkumar, A. (2023). LeanDojo: Theorem proving with retrieval-augmented language models. In *Advances in Neural Information Processing Systems (NeurIPS)*, volume 36. arXiv:2306.15626.

Zhang, C. E., Collins, K. M., Weller, A., and Tenenbaum, J. B. (2023). AI for mathematics: A cognitive science perspective. arXiv:2310.13021.
