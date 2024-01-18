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
