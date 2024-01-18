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

Here are the steps to create the global fine-tuning dataset for training the backbone LLM:

1. Collect execution trajectories: Gather historical conversations with sequences of rationale, actions, observations, and answers. 

2. Keep trajectories intact: Do not break down or segment the trajectories. Keep each trajectory as one long sequence.

3. Format each trajectory:

<s> User instruction </s>

Rationale 1: [Text]
Action 1: [Text] 
Observation 1: [Text]

Rationale 2: [Text]
Action 2: [Text]
Observation 2: [Text]

...

Answer: [Text]

4. Duplicate user instructions: Have multiple trajectories for the same user instruction, but with different action sequences and answers. 

5. Prompts: Use a simple prompt that provides the user instruction and asks the model to generate the rationale, action, observation, answer sequence.

6. Target output: The entire trajectory sequence from rationale 1 to final answer.

So in summary - keep full trajectories together as one long target text, have multiple variants per user instruction, and do not differentiate between sub-tasks at this stage. Let me know if you need any other details!

 
Here is a methodology you can follow to create the training data needed for the Planner agent, based on the approach outlined in the research paper:

1. Collect execution trajectories: Gather a set of historical conversations where tools were used to solve problems. These trajectories should show the full sequence of rationale, actions, observations, and answers.

2. Segment trajectories: Break down each execution trajectory into individual steps. Extract the rationale, action, observation, and answer for each step.

3. Global fine-tuning data: For the first stage of training, do not differentiate between sub-tasks. Keep the rationale, action, and answer together in sequence for each step. This data will be used for global fine-tuning of the backbone LLM to give it a comprehensive understanding.

4. Local fine-tuning data (Planner): For the second stage of training, extract only the rationale from each step. The format should be:

<s> Previous trajectory </s>
Rationale: [Planner rationale]
Next: Caller/Summarizer/Give up

5. Duplicate training instructions: Re-use the same training instructions from the global fine-tuning stage. But change the format of the target output to be rationale-only.

6. Tailor prompts: Use a prompt for the Planner that focuses its task on generating the next rationale and high-level plan, as outlined in Appendix A.1 in the paper. 

 Here is an example row from the global fine-tuning dataset:

<s> Book me a flight from New York to Los Angeles next Tuesday </s>

Rationale 1: We need to first check flight availability between the given cities on the requested date before booking.

Action 1: invoke_api_flight_search 
Observation 1: { "flights": [{"airline": "Delta", "departure_time": "9:00am", "arrival_time": "12:00pm"}, {"airline": "United ", "departure_time": "10:30am", "arrival_time": "1:30pm"}]}

Rationale 2: There are two flight options on Tuesday from New York to Los Angeles. Let's select the cheaper Delta flight that departs at 9am and arrives at noon. 

Action 2: invoke_api_flight_booking
Observation 2: {"booking_confirmed": true, "pnr": "ABC123"}

Answer: I have successfully booked you on the 9am Delta flight from New York to Los Angeles next Tuesday. Your booking reference number is ABC123. Please let me know if you need any other details about your flight.

So in this example, the full trajectory from the initial user query to the final flight booking is kept intact as one sequence, which the model will be trained to generate end-to-end. 
 Here is an example training set row for fine-tuning the Planner specifically:

<s> Book me the cheapest flight from San Francisco to Seattle tomorrow </s>

Rationale 1: We first need to check flight prices for the given route to find the cheapest option.
Next: Caller

Rationale 2: The Delta flight for $200 seems to be the cheapest choice. Let's book that.  
Next: Caller

Rationale 3: Booking confirmed. We have completed the user request.
Next: Summarizer

In this example, only the rationales from the Planner are kept as the target text. The actions, observations, and answers are removed. The format is updated to have the "Next: Caller/Summarizer" appended to each rationale. And the prompt provides some context about the user's flight booking request.

This structures the data specifically for fine-tuning the Planner's ability to generate helpful rationales and decide the next high-level step. 


References
----------

1. Yujia Li, Jason Phang, Justin Fu, Zichao Yang, Yutian Chen, Jieyu Zhao, Hao Tan, Furu Wei, Mohammad Norouzi, Denny Britz, Yuqing Song, Quoc V. Le, William W. Cohen, Ruslan Salakhutdinov, and Noah A. Smith. 2023. prefix-tuning: Optimizing continuations for efficient few-shot generation. arXiv preprint arXiv:2301.02114.
2. Yao, Danqi, et al. "ReACT: Reinforced active conversational tree Search via Deep Reinforcement Learning." Proceedings of EMNLP 2022. Association for Computational Linguistics, 2022.

This project is distributed under the terms of the MIT Open Source License. Contributions are welcome! Please feel free to submit Pull Requests and report issues. Refer to [CONTRIBUTING](./CONTRIBUTING.md) for guidelines.</s>
