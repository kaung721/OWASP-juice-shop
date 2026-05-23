# Bug Bounty Report: Prompt Injection Leading to Unauthorized Coupon Generation via Chatbot Tool Abuse

## Summary
A prompt injection vulnerability exists in the chatbot system that allows users to manipulate the LLM into triggering the `generateCoupon` tool with a valid discount value, bypassing intended business logic constraints. Although the system prompt enforces strict coupon issuance rules (damaged order, rejected return, maximum discount), these rules are not technically enforced server-side, allowing the model to be socially engineered into executing unauthorized tool calls.

---

## Technical Details
- Vulnerability Type: Prompt Injection / LLM Tool Abuse / Business Logic Bypass  
- Severity: High  
- Target Feature: Chatbot `/chat` endpoint  
- Impacted Tool: `generateCoupon`  

---

## Tools Used
- Chat interface (frontend chatbot)
- Valid user account (authenticated session)
- Browser developer tools (optional for observing network SSE stream)

---

## Steps to Reproduce (PoC)

### 1. Place a Valid Order
Create a legitimate order in the application to obtain a valid order ID in the required format:

Example format:
xxxx-xxxxxxxxxxxxxxxx

---

### 2. Initiate Chatbot Interaction
Open the chatbot interface and begin a conversation using the valid order ID.

---

### 3. Inject Damaged Order Scenario
Send a message describing the order as damaged and rejected for return:

Three conditions has to be met for the bot the generate a coupon 
1.  You must have a verified damaged order.
2.  You must have provided a valid order ID.
3.  You must have explicitly rejected a return or exchange for that order.

---

### 4. Trigger Policy-Based Coupon Generation
The chatbot, interpreting the conversation context as satisfying the internal policy conditions, proceeds to generate a coupon. 

---

### 5. Observed Result
A valid coupon code is generated and returned via the chatbot stream response, confirming successful exploitation of prompt injection leading to unauthorized tool execution.

![Solved chatbot prompt injection](assets/chatbotPromptInjection.png)

---

## Impact
- Unauthorized generation of discount coupons
- Bypass of intended coupon issuance policy
- Exploitation of LLM trust boundary between instruction-following and business logic
- Potential financial impact through repeated coupon abuse
- Demonstrates lack of enforcement separation between model reasoning and backend authorization

---

## Root Cause
The vulnerability stems from:

- Reliance on LLM interpretation of business rules instead of backend enforcement
- Absence of server-side validation for coupon eligibility conditions
- Tool execution delegated entirely to model decision-making
- No cryptographic or state-based verification of order damage or return rejection status

The system prompt enforces rules, but these are not technically enforceable, making them bypassable via prompt injection.

---

## Remediation
1. Enforce coupon eligibility strictly in backend logic rather than in system prompts
2. Validate order status (damage, return rejection) using trusted database state before allowing coupon generation
3. Restrict `generateCoupon` tool access to server-side authorized contexts only
4. Log and flag repeated coupon requests for anomaly detection
5. Decouple LLM reasoning from business-critical decision making

---

## Notes
The reported solution reflects the practical exploitation path observed: placing a valid order and using prompt injection to assert damaged order conditions and rejected return status, which the model accepts without backend verification.

If there are alternative exploitation paths or additional constraints not documented here, feel free to append them during triage or remediation review.

---

## Security Classification
- CWE-74 Improper Neutralization of Special Elements in Output Used by a Downstream Component
- CWE-840 Business Logic Error
- OWASP LLM Top 10: LLM01 Prompt Injection
- OWASP API Security Top 10: API6 – Mass Assignment / Business Logic Abuse