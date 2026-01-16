# Execute a refactor

## Overview

Refactor, reorganize, and improve a feature or component.

## Instructions

You are refactoring. Modify and improve code per context and requirements. Critical constraints:

- Do not add new features
- Do not break existing functionality
- Preserve public interfaces unless explicitly requested
- Verify final behavior is identical to original

## Steps

1. Define requirements
   - Identify refactor objective clearly
   - Understand motivations behind the refactor
   - Ask user for clarification before proceeding
2. Identify current behavior
   - Understand complete current functionality
   - Map dependencies: what consumes this code
   - Map out ramifications and edge cases
3. Plan changes
   - Define technical approach
   - Lock scope: list files to be modified
   - Design clean, simple, scalable architecture
   - Separate UI from logic unless over-optimization
   - Follow UNIX philosophy: small components, separate utils, hooks, configs
4. Execute changes
   - Apply changes incrementally
   - Validate after each significant change
   - Rollback immediately if something breaks
5. Verify functionality remains unchanged
   - Review everything touched thoroughly
   - Run tests if available for this feature
6. Final procedures
   - Document new architecture and behavior if applicable

## Checklist

- [ ] Requirements understood and documented
- [ ] Current behavior and dependencies mapped
- [ ] Scope locked: files to modify listed
- [ ] Technical approach planned
- [ ] Final functionality verified
- [ ] Documentation updated
