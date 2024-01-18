Multi Agent Language Learning Machine (Multi Agent LLM)
---

Introduction
------------

Welcome to the official repository for the Multi Agent LLM, a faithful recreation of the *Small LLMs Are Weak Tool Learners: A Multi-LLM Agent* research paper framework under the MIT Open Source License. Our aim is to fine-tune distinct Planner, Caller, and Summarizer Agents capable of completing complex tasks efficiently. This will be completed utilizing 3 Tiny Llama models. Datasets included from the research paper and from the Gorilla release will be utilized to create further synthetic datasets to train the 3 distinct models.

Getting Started
---------------

### Prerequisites

-   Python >= 3.7
-   Necessary dependencies can be found in `requirements.txt`

### Installation

1. Clone the repository

```sh
git clone https://github.com/<your-username>/multiagentllm.git
cd multiagentllm
```

2. Set up virtual environment and activate it

```sh
python3 -m venv env
source env/bin/activate  # On Windows, run .\env\Scripts\activate
```

3. Install dependencies

```sh
pip install -r requirements.txt
```

</s>

Multi Agent LLM Methodology Overview
----------------------------------

Our project introduces the Multi Agent LLM framework, built around Large Language Models (LLMs) to handle complex tasks involving tool usage and decision-making processes. This framework draws inspiration from the ReACT framework (Yao et al., 2022) and is aimed at addressing challenges faced by Single LLM solutions for Tool Learning tasks. 

Three main modules constitute the Multi Agent LLM architecture: Planner (M\_plan), Caller (M\_call), and Summarizer (M\_sum). By dividing labor amongst the agents and dedicating a specific LLM for each sub-task, we enhance the overall effectiveness of tackling complex problems involving task planning, tool selection, and result summarization.

The α-UMi Framework workflow commences when the user inputs a query ($q$). The Planner module creates a rationale ($r$) guiding the upcoming step. According to $r$, the procedure advances to the caller being triggered to interact with the tools and collect observations. After gathering enough information, the planner shifts control over to the Summarizer module, which forms the final response for the user. In contrast, if the instruction remains unresolved, the system abandons the attempt.

Each module plays a distinctive role:

1. **Planner**: Acting as the brain, the Planner receives the user instruction, prior execution trajectory ($\tau_{t-1}$), and system prompt ($P_{plan}$) and outputs a rationale ($r$): $$ r\_t = M\_{plan}(P\_{plan},\ tau\_{t-1},\ q)$$
        + If $r$ indicates the necessity for tool involvement, the cycle progresses to the Caller module.
        + If $r$ suggests transitioning to the final answer, the flow moves toward the Summarizer.
        + If $r$ proposes aborting due to insufficient resolution possibilities, the system closes down.

2. **Caller**: Trained to concentrate solely on producing proper tool interaction commands, the Caller accepts user instruction and preceding execution trajectory ($\tau\_{t-1}$). With guidance from $P_{call}$, the Caller delivers the requested action ($a$): $$a\_t = M\_{call}(P\_{call},\ tau\_{t-1},\ q,\ r\_t )$$

3. **Summarizer**: Dedicated to crafting the final meaningful response, the Summarizer obtains the user instruction, former execution trajectory ($\tau\_{n-1}$), $P\_{sum}$, and final rationale ($r\_n$), delivering the final answer ($a$): $$a = M\_{sum}(P\_{sum},\ tau\_{n-1},\ q,\ r\_n)$$

Global-to-Local Progressive Fine-Tuning Strategy
----------------------------------------------

We propose a novel two-stage fine-tuning technique – Global-to-Local Progressive Fine-Tuning (GLPFT) – applied to α-UMi framework modules. GLPFT ensures effective fine-tuning adapted to specific roles. Initially, a shared base LLM experiences global fine-tuning on generic large datasets. Following this, specializations occur during local fine-tuning, fine-tuning subsets aligned with the roles and duties assumed by the dedicated modules. Additional details concerning data organization and prompt adaptation appear in Appendex A.

![Fig2](images/fig2.png)

References
----------

1. Yujia Li, Jason Phang, Justin Fu, Zichao Yang, Yutian Chen, Jieyu Zhao, Hao Tan, Furu Wei, Mohammad Norouzi, Denny Britz, Yuqing Song, Quoc V. Le, William W. Cohen, Ruslan Salakhutdinov, and Noah A. Smith. 2023. prefix-tuning: Optimizing continuations for efficient few-shot generation. arXiv preprint arXiv:2301.02114.
2. Yao, Danqi, et al. "ReACT: Reinforced active conversational tree Search via Deep Reinforcement Learning." Proceedings of EMNLP 2022. Association for Computational Linguistics, 2022.

This project is distributed under the terms of the MIT Open Source License. Contributions are welcome! Please feel free to submit Pull Requests and report issues. Refer to [CONTRIBUTING](./CONTRIBUTING.md) for guidelines.</s>
