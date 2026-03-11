<!-- WHEN TO USE: Use this when you need a deep security analysis of specific modules or the entire codebase. -->
<!-- FILL IN: all [PLACEHOLDER: description] values before dispatching -->

## Task
Audit the codebase for security vulnerabilities, focusing on [PLACEHOLDER: specific modules or concerns]. Perform a comprehensive check against OWASP Top 10 categories and project-specific risk vectors.

## Analytical Dimensions
Evaluate the loaded files across these critical security dimensions:
1. **Injection Risks:** Check for unsanitized user input in SQL queries, OS commands, or HTML templates.
2. **Broken Authentication:** Look for weak session management, hardcoded credentials, or bypassable login logic.
3. **Sensitive Data Exposure:** Identify unencrypted storage of PII, secrets in logs, or insecure transport.
4. **Broken Access Control:** Verify that authorization checks are enforced on every sensitive operation/endpoint.
5. **Security Misconfiguration:** Check for overly permissive CORS, debug modes enabled in production, or insecure defaults.
6. **Cross-Site Scripting (XSS):** Identify improper escaping of user-generated content in the UI.
7. **Insecure Deserialization:** Look for untrusted data being deserialized into objects.

## Files to Load
Load ALL of these files at once using read_many_files before starting:
- [PLACEHOLDER: absolute path to relevant file]
- [PLACEHOLDER: add more paths]

## Search Patterns
Use ripgrep to identify potential "hot spots" for manual review:
- **Hardcoded Secrets:** `(api[_-]?key|secret|password|token|auth)\s*[:=]\s*['"][a-zA-Z0-9]{8,}['"]`
- **Command Injection:** `\b(eval|exec|system|spawn|process\.shell)\s*\(`
- **SQL Injection (String Concatenation):** `\.(query|execute|run)\s*\(\s*[`'"].*?\$\{.*?\}['"]`
- **Insecure UI (React/Vue):** `dangerouslySetInnerHTML|v-html`
- **Missing Auth/Permission Hooks:** `\b(useAuth|isAuthenticated|requireAdmin)\b` (Search for absence in protected modules)

## Constraints
- Read and analyze files only — do not modify source code
- Write output only to the path specified in the ## Output section
- Do not ask for confirmation before writing the output file

## Output
Write result to: [PLACEHOLDER: <project-root>/.ai-team/outputs/gemini-<task-id>-output.md]
Format: markdown report with the following sections:
1. **Executive Summary:** High-level risk posture.
2. **Vulnerability Table:** Categorized by Severity (Critical, High, Medium, Low).
3. **Detailed Findings:** For each issue, include:
   - **Description:** What is the vulnerability?
   - **Location:** File path and line numbers.
   - **Proof of Concept:** Example of how it could be exploited.
   - **Remediation Options:** Technical approaches with trade-offs to address the vulnerability, for Coordinator review.
4. **Candidate Improvements:** Potential process-level changes or tools for the Coordinator to evaluate (e.g., "Consider a linter rule for X").

## Done When
Output file exists at the specified path

## BEFORE YOU EXIT
Your final action MUST be writing the output file to the .ai-team/outputs/ path above. Do not ask for confirmation.
