# ai-coding-agent-skills
Practical skills and workflows for using AI coding agents safely and effectively.

## Core practical skills

- Write precise prompts with clear goals, constraints, and acceptance criteria.
- Break work into small, reviewable steps to reduce risk.
- Ask the agent to explain assumptions before implementing changes.
- Require tests or checks for every meaningful code change.
- Review diffs critically instead of trusting generated code blindly.

## Safe and effective workflow

1. **Plan first**: define scope, risks, and done criteria.
2. **Inspect context**: read relevant files, tests, and build setup.
3. **Change minimally**: prefer surgical edits over broad rewrites.
4. **Validate early**: run targeted tests/checks after each change.
5. **Review for security**: check for secrets, unsafe input handling, and dependency risk.
6. **Document outcomes**: summarize what changed and why.
