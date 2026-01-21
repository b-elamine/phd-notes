LLM, DevSecOps, Cloud Security

**Short overview**

The authors build an instruction-tuning dataset, GenSIaC, that explicitly teaches models to recognize common IaC security weaknesses during generation and inspection. With lightweight LoRA fine-tuning,.


- Build a dataset (**GenSIaC**) with examples of **insecure IaC configurations, explanations of why they are insecure, and secure versions**.
    
- They **fine-tune the LLM on these security-focused instructions**, so the model learns security rules and patterns during training.
    
- After training, the model **automatically avoids common security mistakes while generating IaC**, without needing external security scanners or rule engines.