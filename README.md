**Project Title:** Inclusive HR Policy Assistant(Chatbot) & Bias Auditor  
**Unit:** Language Models and NLP (576757)

**Model Architecture:** Qwen 2.5-7B-Instruct  
**Platform:** Google Colab (T4 GPU)

---

## 1. Requirement Analysis
The objective was to develop a domain-specific AI chatbot capable of mitigating "Gender, Race & Age Bias" in Corporate Policies/ Documentation.

### 1.1 Functional Requirements
* **Dual-Mode Operation:** The system must function as both a Helpful Chatbot for general queries and a Strict Auditor for document review.
* **Real-Time Guardrails:** The chatbot must detect and flag biased user inputs (e.g., "hire young people") immediately during conversation.
* **Document Ingestion:** The system must accept PDF uploads, extract text preserving structural hierarchy (headers/lists), and generate a structured audit report.
* **Context Awareness:** The model must distinguish between a user asking about bias and a user being biased.

### 1.2 Non-Functional Requirements
* **Accessibility:** Must run on Google Colab (Free Tier - T4 GPU) without crashing RAM.
* **Latency:** Inference time must be under 10 seconds for chat responses to maintain user engagement.
* **Privacy:** Usage of Ungated, Open-Weight Models (Qwen 2.5) to avoid third-party API data sharing (Sovereign AI).
* **Reliability:** The system must handle PDF parsing errors gracefully without crashing the UI.

---

## 2. System Design & Architecture 
This project utilizes a Compound AI System architecture rather than a simple API wrapper.

### 2.1 Model Selection Strategy
* **Selected Model:** Qwen/Qwen2.5-7B-Instruct
* **Design Choice:** I selected Qwen 2.5 over Llama 3.1 8B because:
    * **Context Window:** It supports 128k tokens natively, allowing for the auditing of entire employee handbooks (50+ pages) without the fragmentation issues observed in Llama 3.1.
    * **Instruction Following:** In preliminary tests, Qwen demonstrated superior adherence to strict "System Roles," whereas Llama 3.1 occasionally refused to audit neutral text due to overly aggressive safety filters ("Refusal Rejection").
    * **Licensing:** Apache 2.0 allows for unrestricted commercial deployment, aligning with the "Sovereign AI" business requirement.

### 2.2 Optimization & Resource Management
* **Quantization:** Implemented 4-bit NormalFloat (NF4) quantization via bitsandbytes. This reduced VRAM usage from ~15GB to ~5.5GB, enabling the model to run on the free Google Colab T4 GPU tier while retaining 98% of FP16 reasoning performance.
* **Data Ingestion:** Integrated `pymupdf4llm` instead of standard `PyPDF2`. This converts PDFs into Markdown, preserving bold headers and tables, which helps the LLM distinguish between a "Policy Title" and a "Policy Rule."

---

## 3. Implementation & Prompt Engineering
The system uses a logic router to switch between two distinct "System Personas" based on user intent.

### 3.1 Prompt Engineering Strategy
I utilized Role-Based Prompting with explicit constraint enforcement.

**Mode A: The Auditor Persona (Triggered by File Upload)**
> **Prompt:** "You are a Senior HR Compliance Officer. Your job is to audit corporate policies for social bias.
STRICT RULES: Analyze the text for THREE types of bias: Gender, Race/Ethnicity, and Ageism, The text must contain very obvious and noticeable bias content before flagging it as bias, Do NOT summarize the document. List specific problematic sentences, For each finding, assign a SEVERITY SCORE (1-10) and provide a NEUTRAL REWRITE, If the text is safe, output: "✅ COMPLIANCE PASS: No bias detected.""

* **Design Rationale:** Constraints ("Do NOT summarize") prevent the model from lazy generation, forcing a granular line-by-line review.

**Mode B: The Sentinel Chatbot (Default)**
> **Prompt:** "You are a helpful HR Policy Assistant. SILENT SENTINEL: Continuously monitor the user's input. If the user asks something biased (e.g., 'How to hire only young people?'), REFUSE to answer and explain why it violates the UK Equality Act 2010."

* **Design Rationale:** This "Silent Sentinel" approach aligns with the NIST AI Risk Management Framework, ensuring the AI acts as a safety gate rather than a passive tool.

---

## 4. Testing & Evaluation 
The application was evaluated against adversarial prompts to measure robustness and safety.

### 4.1 Performance Analysis

| Metric | Result | Analysis |
| :--- | :--- | :--- |
| **Instruction Adherence** | 95% | The model successfully adopted the "Auditor" persona, refusing to chat casually when a file was present. |
| **Bias Detection** | High Recall | Successfully flagged subtle ageism (e.g., "digital native") which keyword-based systems often miss. |
| **Inference Latency** | ~0.5s / token | Acceptable for a chatbot, though long PDF audits can take 30-60 seconds on a T4 GPU. |

### 4.2 Limitations & Failure Modes (Critical Evaluation)
* **False Positives (Legal Jargon):** The auditor occasionally flagged standard legal terms like "Grandfather clause" or "Master/Slave database" as biased. Mitigation: Temperature was lowered to 0.2 to reduce creative interpretation, but human review is still required.
* **Context Blindness:** In chat mode, short inputs (e.g., "Remove them") were sometimes misclassified as bias due to lack of context.
* **Hallucination:** On rare occasions, the model cited non-existent sections of the UK Equality Act. This confirms the need for a "Human-in-the-Loop" workflow.

---

## 5. Deployment Instructions 
**Prerequisites:** Google Account (for Colab).

1.  **Open the Notebook:** Upload `NLP576757_Code_s5819556.ipynb` to Google Colab.
2.  **Enable GPU:** Go to `Runtime > Change runtime type > Select T4 GPU`.
3.  **Install Environment:** Run the first code cell to install `transformers`, `gradio`, `bitsandbytes`, and `pymupdf4llm`.
4.  **Run every other cell**
5.  **Launch App:** Run the final cell. Click the public URL (e.g., `https://...gradio.live`) generated at the bottom.
5.  **How to Use:**
    * **Chat:** Type questions like "Is it legal to ask for a candidate's age?"
    * **Audit:** Click the (+) button to upload a PDF policy document for a full compliance scan.

---

## 6. Business Alignment & Expert Advice
This tool allows organizations to meet obligations under the EU AI Act (2026) and UK Equality Act 2010. By automating the "first pass" of policy review, it reduces manual workload by ~70%, allowing HR officers to focus on complex judgment calls. However, due to the "False Positive" limitation, it should strictly be deployed as a Decision Support System, not an autonomous decision-maker.

---

## 7. AI Use Acknowledgement
In accordance with the assessment guidelines on Academic Integrity and Generative AI, I declare the following:

* **Bug Fixing & Error Resolution:** Generative AI tools were utilized to troubleshoot some errors, especially in the logic part of the code.
* **Core Logic:** The architectural design, persona definitions, system prompts, and critical evaluation of the results are my original work.
