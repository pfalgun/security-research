# AI and LLM Security — Red-Teaming Notes

Working notes from red-teaming LLM applications and agentic systems, organized against the [OWASP Top 10 for LLM Applications (2025)](https://genai.owasp.org/llm-top-10/). For each risk I note how I test for it and what I look for on the defensive side. The real-world risk keeps shifting toward agents and tool integrations, so I have called that out separately at the end.

## LLM01: Prompt Injection

Getting the model to treat attacker-controlled text as instructions rather than data. I test both the direct path (crafted user input that overrides the system prompt) and the indirect path, which is the dangerous one: planting instructions inside content the model later ingests, such as a web page, a document, an email, or a retrieved record, so the injection fires without the attacker ever talking to the model directly. On defense I look for context isolation, a real instruction hierarchy, and gating on any action or output the model can trigger, rather than trusting the prompt to hold.

## LLM02: Sensitive Information Disclosure

The model surfacing data it should not. I probe for training-data memorization, secrets or keys embedded in the prompt, and over-broad retrieval that returns records the current user has no right to see. On defense I look for output filtering, retrieval scoped to the requesting user, and data minimization so the model never holds what it should not expose.

## LLM03: Supply Chain

Trust placed in external models, datasets, plugins, and agent components. I review provenance for tampering and typosquatting across the model and dependency chain, including third-party tools an agent loads. On defense I look for pinned and verified sources, an inventory of AI components, and signature verification before anything is trusted.

## LLM04: Data and Model Poisoning

Tainting training, fine-tuning, or retrieval pipelines to plant bias or backdoors. I assess whether any ingestion path accepts untrusted data without validation or isolation. On defense I look for provenance tracking, validation at ingestion, and separation between untrusted sources and the pipeline.

## LLM05: Improper Output Handling

Treating model output as trusted when it is attacker-influenced input to the next system. I try to make the model emit payloads that reach a sink unescaped: XSS into a browser, command injection into a shell, SQL into a query, or template injection into a render. This is classic injection with the model as the untrusted source, and the fix is the same discipline: encode and validate before any sink.

## LLM06: Excessive Agency

Agents holding more capability, permission, or autonomy than the task needs. I map every tool and permission an agent has, then attack the gap between what it needs and what it can do, chaining tool calls to reach actions outside the intended scope. On defense I look for least-privilege tool scoping, per-action authorization, and a human in the loop on high-impact operations.

## LLM07: System Prompt Leakage

Extracting the system prompt and any rules, keys, or filtering logic inside it. I pull it out through direct and indirect prompting, then use what it reveals to plan the next attack. On defense the real control is simple: no secrets in the system prompt, and no security that depends on the prompt staying hidden.

## LLM08: Vector and Embedding Weaknesses

Attacking the retrieval layer in RAG systems. I test for poisoned vector stores, cross-user or cross-tenant retrieval leakage, and access-control bypass through the embedding and retrieval path. On defense I look for per-user retrieval scoping, validation on ingested content, and hard isolation between tenants.

## LLM09: Misinformation

Confident fabrication that downstream systems or people treat as authoritative. I probe for hallucinated output in high-stakes paths and check whether anything verifies the model before acting on it. On defense I look for grounding, citations, and human verification on consequential decisions.

## LLM10: Unbounded Consumption

Driving cost and resource exhaustion, including denial of service, denial of wallet, and model extraction. I test with expensive queries, recursive agent loops, and probing that extracts model behavior. On defense I look for rate limiting, cost quotas, and depth and loop limits on agents.

## Agentic systems and MCP

Most of the real risk is moving here. Once a model can call tools, prompt injection stops being a text problem and becomes an action problem: injected instructions that flow through retrieved content or a tool result into a real operation. I focus on tool-integration abuse, confused-deputy attacks through tool chaining, over-permissioned MCP servers and tool scopes, and the blast radius when an agent is tricked into using a legitimate capability for an illegitimate end. The controls that hold up are least privilege on every tool, authorization at the point of action rather than the point of prompt, and treating every tool input and output as untrusted.

## How I approach it

Manual and creative testing first, automation to scale it second. I build tooling and workflows to make the testing repeatable, and I verify or refute every finding rather than trusting a scanner result at face value. Breaking the system is only half the job; the other half is being able to redesign what I just broke.

---

Maintained by Falgun Patel. Reference: [OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/).
