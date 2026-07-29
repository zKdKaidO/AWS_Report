# Documentation Rules

## Writing requirements

1. Write in clear technical English.
2. Use the project facts in PROJECT_CONTEXT.md as the source of truth.
3. Do not invent:
   - AWS resource names
   - test results
   - performance numbers
   - monthly costs
   - screenshots
   - commands that were not executed
4. Mark missing evidence as:
   `Evidence required`
5. Distinguish clearly between:
   - Implemented
   - Verified
   - Proposed
   - Optional production hardening
6. Do not describe proposed services as already deployed.
7. Preserve the existing Hugo front matter.
8. Use relative links between workshop pages.
9. Include commands in fenced bash blocks.
10. Include expected output after important commands.
11. Do not place secrets, passwords, database URLs or tokens in documentation.
12. Use placeholders such as:
    - `<DATABASE_ENDPOINT>`
    - `<SECRET_NAME>`
    - `<EMAIL_ADDRESS>`
13. Every workshop page should contain:
    - Objective
    - Architecture context
    - Steps
    - Verification
    - Expected result
    - Common errors
    - Outcome
14. Mermaid diagrams must be valid for draw.io import.
15. Cost values must not be invented. Use:
    - AWS Pricing Calculator results supplied by the user
    - or clearly marked estimates with assumptions
16. Self-evaluation must be written in first person.
17. Feedback must not fabricate comments from supervisors.
18. Any missing supervisor feedback must remain a placeholder.