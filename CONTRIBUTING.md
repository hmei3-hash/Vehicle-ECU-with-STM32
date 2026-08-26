# Contributing — Personal Development Workflow

This is a personal learning project. This document defines the git workflow and development process.

## Branch Strategy

```
main (stable, tested code only)
 │
 ├── feature/phase-0-hardware-bringup
 ├── feature/phase-1-gpio
 ├── feature/phase-2-timer-interrupt
 ├── feature/phase-3-sensor-driver
 ├── feature/phase-4-can-basic
 ├── feature/phase-5-multi-node-can
 ├── feature/phase-6-can-protocol
 ├── feature/phase-7-freertos
 ├── feature/phase-8-ecu-architecture
 ├── feature/phase-9-fault-detection
 ├── feature/phase-10-testing
 └── feature/phase-11-documentation
```

## Workflow for Each Phase

1. **Create a feature branch:**
   ```
   git checkout -b feature/phase-X-description
   ```

2. **Implement the phase yourself:**
   - Write the code
   - Test on hardware
   - Debug and iterate

3. **Test:**
   - Run the relevant test cases from TEST_PLAN.md
   - Record actual results and pass/fail status
   - Capture logic analyzer traces if applicable

4. **Commit with meaningful messages:**
   ```
   git add .
   git commit -m "phase-X: brief description of what you implemented"
   ```
   
   Commit message examples:
   - `phase-1: configure GPIO PA5 as output for LED`
   - `phase-4: implement CAN TX with loopback test`
   - `phase-7: add FreeRTOS sensor read task`

5. **Document your work:**
   - Update code comments
   - Fill in test results in TEST_PLAN.md
   - Update DEVELOPMENT_PLAN.md checkboxes

6. **Merge to main:**
   ```
   git checkout main
   git merge feature/phase-X-description
   ```

## Commit Guidelines

- Commit often — each meaningful step should be a commit
- Write commit messages in imperative mood: "add", "implement", "fix", "configure"
- Prefix with the phase number: `phase-X: message`
- Never commit generated/binary files (add to .gitignore)

## Code Style

- [TBD — define your C code style: indentation, naming conventions, etc.]
- Comment every register configuration with WHY, not just WHAT
- Every function should have a brief header comment describing its purpose

## When to Ask for Help

- After you've attempted implementation and are stuck
- When you want a code review after completing a phase
- When you need to understand a concept before implementing
- When debugging a hardware issue

## What NOT to Do

- Do not copy code from examples without understanding it
- Do not skip testing
- Do not merge untested code to main
- Do not leave [TODO] items unresolved when merging a phase
