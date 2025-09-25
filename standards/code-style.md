# Code Style Guide

## Context

Code style rules for the Palma Android Application project.

<conditional-block context-check="general-formatting">
IF this General Formatting section already read in current context:
  SKIP: Re-reading this section
  NOTE: "Using General Formatting rules already in context"
ELSE:
  READ: The following formatting rules

## General Formatting

### Indentation
- Use 4 spaces for indentation (never tabs)
- Maintain consistent indentation throughout files
- Align nested structures for readability

### Naming Conventions
- **Methods and Variables**: Use camelCase (e.g., `userProfile`, `calculateTotal`)
- **Classes and Modules**: Use PascalCase (e.g., `UserProfile`, `PaymentProcessor`)
- **Constants**: Use UPPER_SNAKE_CASE (e.g., `MAX_RETRY_COUNT`)

### String Formatting
- Use double quotes for strings: `"Hello World"`
- Use string templates for interpolation: `"Hello $name"`
- Use triple quotes for multi-line strings

### Code Comments
- Add brief comments above non-obvious business logic
- Document complex algorithms or calculations
- Explain the "why" behind implementation choices
- Never remove existing comments unless removing the associated code
- Update comments when modifying code to maintain accuracy
- Keep comments concise and relevant
</conditional-block>


<conditional-block task-condition="kotlin" context-check="kotlin-style">
IF current task involves writing or updating Kotlin code:
  IF kotlin-style.md already in context:
    SKIP: Re-reading this file
    NOTE: "Using Kotlin style guide already in context"
  ELSE:
    <context_fetcher_strategy>
      IF current agent is Claude Code AND context-fetcher agent exists:
        USE: @agent:context-fetcher
        REQUEST: "Get Kotlin style rules from code-style/kotlin-style.md"
        PROCESS: Returned style rules
      ELSE:
        READ: @.agent-os/standards/code-style/kotlin-style.md
    </context_fetcher_strategy>
ELSE:
  SKIP: Kotlin style guide not relevant to current task
</conditional-block>

<conditional-block task-condition="jetpack-compose" context-check="compose-style">
IF current task involves writing or updating Jetpack Compose UI code:
  IF jetpack-compose-style.md already in context:
    SKIP: Re-reading this file
    NOTE: "Using Jetpack Compose style guide already in context"
  ELSE:
    <context_fetcher_strategy>
      IF current agent is Claude Code AND context-fetcher agent exists:
        USE: @agent:context-fetcher
        REQUEST: "Get Jetpack Compose style rules from code-style/jetpack-compose-style.md"
        PROCESS: Returned style rules
      ELSE:
        READ: @.agent-os/standards/code-style/jetpack-compose-style.md
    </context_fetcher_strategy>
ELSE:
  SKIP: Jetpack Compose style guide not relevant to current task
</conditional-block>
