# AI-Assisted Hackathon Toolkit

**From problem discovery to a secure, working MVP.**

Welcome! This repository contains a collection of structured AI prompts designed to help hackathon participants turn ideas into viable app-based solutions using generative AI and vibe coding.

The goal is simple: **build something people need, validate that the opportunity is real, and engineer a solution that works beyond a convincing demo.**

## What's Inside?

### 1. Idea Discovery & Market Validation

Use this prompt to move beyond the first idea that comes to mind.

It guides you through:
- Understanding the problem and identifying target users.
- Researching competitors and comparing their features.
- Identifying underserved users and market gaps.
- Finding empirical evidence to validate the opportunity.
- Generating and ranking three differentiated app ideas.
- Estimating revenue potential, build complexity, and commercial viability.
- Defining a realistic hackathon MVP and measurable success criteria.

**Outcome:** An evidence-informed app concept with a clear value proposition and a reason to exist.

### 2. System Architecture & Engineering Blueprint

Use this prompt once you have selected your app idea, before generating the code.

It guides you through:
- Selecting appropriate technologies, APIs, and services.
- Designing a system architecture diagram using Mermaid.
- Structuring a modular, reusable, maintainable codebase.
- Defining API contracts, data models, and component responsibilities.
- Applying secure-by-design principles.
- Planning for failures, invalid inputs, unavailable services, and unexpected errors.
- Establishing unit, integration, end-to-end, and security testing.
- Planning deployment, configuration, monitoring, and recovery.
- Breaking implementation into small, testable increments.

**Outcome:** A practical engineering blueprint for building the MVP with clear boundaries, testable components, and appropriate security controls.

## Recommended Workflow

Follow these prompts in order.

1. **Discover:** Use the Idea Discovery prompt with your hackathon problem statement.
2. **Validate:** Research the market, verify sources, and assess whether the proposed opportunity is credible.
3. **Select:** Choose the idea that best balances user value, differentiation, feasibility, and commercial potential.
4. **Architect:** Use the Engineering Blueprint prompt to design the system before writing code.
5. **Build:** Implement the core user journey in small, independently testable increments.
6. **Verify:** Test normal behaviour, edge cases, security controls, and failure scenarios.
7. **Deploy:** Publish a working MVP and verify it in the actual demonstration environment.
8. **Demonstrate:** Explain the problem, evidence, solution, technical decisions, limitations, and next steps.

## How to Use the Prompts

1. Open the relevant prompt file in this repository.
2. Copy the complete prompt into your preferred AI assistant or coding tool.
3. Replace the placeholders with your problem statement, app idea, team constraints, and other relevant context.
4. Review the AI's output critically. Verify sources, assumptions, technical compatibility, and cost estimates.
5. Refine the output through follow-up questions before proceeding.

**Tip:** Give your AI assistant enough context to make good decisions, but keep the scope focused. A clearly defined problem and a small, demonstrable MVP are more useful than a long list of ambitious features.

## Engineering Principles

These prompts encourage participants to apply professional engineering judgement while benefiting from AI-assisted development.

- **Problem first:** Solve a real user problem rather than adding technology for its own sake.
- **Evidence over assumptions:** Support market claims with credible sources and distinguish facts from projections.
- **Simplicity over complexity:** Choose the simplest architecture that meets the requirements.
- **Security by design:** Protect user data, enforce authorisation, validate inputs, and manage secrets appropriately.
- **Modularity and reuse:** Separate responsibilities so components can be tested, maintained, and replaced independently.
- **Fail safely:** Handle errors without exposing sensitive information, corrupting data, or falsely reporting success.
- **Test continuously:** Treat AI-generated code as unverified until it has been reviewed and tested.
- **Build incrementally:** Deliver the core user journey before adding optional features.
- **Honest demonstrations:** Clearly distinguish working functionality from mocked services and unfinished production capabilities.

## Important Considerations

These prompts are starting points, not substitutes for research, engineering judgement, or security review.

- AI-generated market research and competitor comparisons must be independently verified.
- Revenue estimates are illustrative projections, not guaranteed outcomes.
- Technology and service recommendations should be checked against current documentation, pricing, and limitations.
- Prototypes may not be suitable for real users or sensitive data without further hardening.
- Applications handling personal information must consider applicable privacy and regulatory obligations, including South Africa's POPIA where relevant.
- Never commit credentials, API keys, passwords, or other secrets to the repository.
- Do not claim that tests passed or that a system is secure unless the relevant checks have actually been performed.

## Who Is This For?

- Hackathon participants and student developers.
- Vibe coders using AI to build applications.
- Founders validating early-stage product ideas.
- Developers who want AI-generated code to follow better engineering practices.
- Mentors helping teams balance innovation, feasibility, security, and business value.

## Final Word

AI can accelerate development, but speed without direction creates technical debt. A promising idea without evidence creates business risk. A working demo without appropriate safeguards creates security risk.

Use these prompts to connect **product thinking, responsible AI use, and sound software engineering**.

Build less, validate more, test what matters, and make every line of code serve a purpose.

