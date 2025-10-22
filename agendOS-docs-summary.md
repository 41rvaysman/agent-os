# Agent OS Documentation Summary

## Overview

Agent OS is a system for spec-driven development with AI agents that ensures consistent, high-quality code matching your conventions. Version 2.1+ introduces Claude Code Skills and Subagents, replacing the older roles system.

**Key Concepts:**
- **Standards:** Markdown files defining coding conventions and patterns
- **Profiles:** Configuration packages bundling standards, workflows, and settings for different tech stacks
- **Skills:** (v2.1+) Intelligent, context-aware application of standards
- **Subagents:** (v2.1+) Specialized AI agents for complex feature implementation
- **Workflows:** Reusable process templates for consistent development patterns

---

## Quick Reference

### Standards Best Practices
✅ **DO:**
- Be specific and actionable with code examples
- Keep standards modular (one concern per file)
- Start minimal and grow organically
- Include the "why" behind conventions

❌ **DON'T:**
- Write vague guidelines like "write clean code"
- Create monolithic standards files
- Skip examples and explanations

### Profile Structure
```
~/agent-os/profiles/[profile-name]/
├── profile-config.yml              # Inheritance & configuration
└── standards/
    ├── backend/                    # Server-side conventions
    ├── frontend/                   # Client-side conventions
    ├── global/                     # Cross-cutting standards
    └── testing/                    # QA patterns
```

### Configuration Options (v2.1+)
```yaml
# ~/agent-os/config.yml or profile-config.yml
claude_code_commands: true                  # Enable Claude Code slash commands
use_claude_code_subagents: true            # Enable specialized subagents
standards_as_claude_code_skills: true      # Convert standards to Skills
```

### Development Workflow
1. `/plan-product` - Define product vision, roadmap, tech stack (one-time)
2. `/shape-spec` - Research and explore feature requirements
3. `/write-spec` - Create detailed specification
4. `/create-tasks` - Break spec into actionable tasks
5. `/implement-tasks` - Simple implementation (smaller features)
   OR `/orchestrate-tasks` - Delegated implementation (complex features)
6. Review & refine standards based on results

---

## Android Profile Analysis

### Profile Overview
The Android profile (`~/agent-os/profiles/android/`) is a custom AgentOS profile for native Android development with Jetpack Compose, demonstrating excellent adherence to AgentOS best practices.

**Location:** [profiles/android/](profiles/android/)
**Configuration:** [profiles/android/profile-config.yml](profiles/android/profile-config.yml)

### Current Structure
```
profiles/android/
├── profile-config.yml
└── standards/
    ├── backend/
    │   ├── datastore.md
    │   ├── dependency-injection.md
    │   ├── repository-pattern.md
    │   └── viewmodel-pattern.md
    ├── frontend/
    │   ├── compose-components.md
    │   ├── navigation.md
    │   └── theming.md
    ├── global/
    │   ├── coding-style.md
    │   ├── conventions.md
    │   ├── coroutines.md
    │   ├── project-structure.md
    │   └── tech-stack.md
    └── testing/
        ├── ui-testing.md
        └── unit-testing.md
```

### Strengths ✅

**1. Proper Profile Inheritance**
- Inherits from `default` profile
- Excludes only standards being replaced (not duplicating)
- Follows layered customization approach

**2. Skills Integration (v2.1+)**
- Enables `standards_as_claude_code_skills: true`
- Reduces token consumption
- Intelligent, context-aware standard application

**3. Modular, Focused Standards**
- One concern per file
- Well-organized by domain
- Easy to maintain and update

**4. Specific and Actionable**
- Every standard includes code examples
- Clear "do" and "don't" patterns
- Explains rationale

**5. Comprehensive Tech Stack Documentation**
- Specifies all dependency versions
- Documents multi-module architecture
- Clear technology choices

**6. Android-Specific Best Practices**
- Jetpack Compose patterns
- MVVM with UiState/UiEvent
- Hilt dependency injection
- Coroutine usage patterns

### Recommended Improvements 🎯

#### Priority 1: Error Handling Standard
**Create:** `profiles/android/standards/global/error-handling.md`

**Should Cover:**
- Exception handling patterns
- Error state representation in UiState
- Logging strategies (e.g., Timber)
- User-facing error messages
- Retry mechanisms
- Network error handling

**Example Pattern:**
```kotlin
// Error state in ViewModel
data class UiState(
    val data: List<Item> = emptyList(),
    val isLoading: Boolean = false,
    val error: ErrorState? = null
)

sealed interface ErrorState {
    data class NetworkError(val message: String) : ErrorState
    data class ValidationError(val field: String) : ErrorState
    data object UnknownError : ErrorState
}

// Error handling in ViewModel
private fun handleError(exception: Exception) {
    _uiState.update { it.copy(
        isLoading = false,
        error = when (exception) {
            is IOException -> ErrorState.NetworkError(exception.message ?: "Network error")
            is ValidationException -> ErrorState.ValidationError(exception.field, exception.message)
            else -> ErrorState.UnknownError
        }
    )}
}
```

#### Priority 2: Performance Best Practices
**Create:** `profiles/android/standards/global/performance.md`

**Should Cover:**
- Compose recomposition optimization
- `remember`, `rememberSaveable`, `derivedStateOf` usage
- LaunchedEffect vs SideEffect vs DisposableEffect
- Memory leak prevention (ViewModel, coroutines)
- Background work scheduling (WorkManager)
- Image loading and caching
- Database query optimization

**Example:**
```kotlin
// ✅ Good - Optimized recomposition
@Composable
fun ExpensiveList(items: List<Item>) {
    val sortedItems = remember(items) {
        items.sortedBy { it.name }
    }

    LazyColumn {
        items(sortedItems, key = { it.id }) { item ->
            ItemRow(item)
        }
    }
}

// ❌ Bad - Recomputes every recomposition
@Composable
fun ExpensiveList(items: List<Item>) {
    val sortedItems = items.sortedBy { it.name }
    // Sorting happens on every recomposition
}
```

#### Priority 3: Expand Testing Standards

**Enhance:** `profiles/android/standards/testing/ui-testing.md`
- Compose testing patterns with ComposeTestRule
- Test tag conventions and semantic properties
- Screenshot testing
- Accessibility testing (TalkBack, content descriptions)

**Add:** `profiles/android/standards/testing/integration-testing.md`
- Repository integration tests
- ViewModel integration tests
- Flow testing patterns
- Database integration tests

**Add Coverage Requirements:**
- Minimum coverage thresholds (e.g., 80% for ViewModels, 70% for repositories)
- What to prioritize (business logic, state management)
- What can be skipped (simple UI components, generated code)

#### Priority 4: Accessibility Standard
**Create:** `profiles/android/standards/frontend/accessibility.md`

**Should Cover:**
- Content descriptions for images and icons
- Semantic properties for custom components
- TalkBack testing procedures
- Touch target sizes (minimum 48dp)
- Color contrast requirements
- Accessible state announcements

#### Priority 5: Networking/API Standard
**Create:** `profiles/android/standards/backend/networking.md`

**Should Cover:**
- HTTP client setup (Retrofit/Ktor patterns)
- API response handling and parsing
- Request/Response model conventions
- Timeout and retry policies
- Authentication token management
- Error response handling

#### Priority 6: Security/Privacy Standard
**Create:** `profiles/android/standards/global/security.md`

**Should Cover:**
- DataStore encryption for sensitive data
- Secure credential storage (KeyStore)
- ProGuard/R8 rules for obfuscation
- API key management (BuildConfig, local.properties)
- Permission handling best practices
- Data sanitization and validation

### Action Items

**Immediate:**
1. Create `error-handling.md` (highest priority)
2. Create `performance.md` (second priority)
3. Run `/improve-skills` after adding new standards

**Short-term:**
2. Expand testing standards (ui-testing, add integration-testing)
3. Add `accessibility.md` for inclusive design
4. Create `networking.md` if using APIs

**Long-term:**
5. Add `security.md` for sensitive data handling
6. Iterate based on real feature implementations
7. Update standards when patterns emerge

### Compliance Summary

| AgentOS Best Practice | Status | Notes |
|----------------------|--------|-------|
| Profile Inheritance | ✅ Excellent | Properly inherits from default |
| Modular Standards | ✅ Excellent | One concern per file |
| Specific Examples | ✅ Excellent | All standards include code examples |
| Tech Stack Documented | ✅ Excellent | Comprehensive tech-stack.md |
| Skills Integration | ✅ Excellent | v2.1 Skills enabled |
| Testing Standards | ⚠️ Good | Could expand (integration, coverage) |
| Error Handling | ❌ Missing | **Need error-handling.md** |
| Performance Guidelines | ❌ Missing | **Need performance.md** |
| Accessibility | ❌ Missing | **Need accessibility.md** |
| Security/Privacy | ⚠️ Partial | Consider adding security.md |

**Overall Grade: A-**

The Android profile demonstrates excellent AgentOS practices with specific, actionable standards and proper configuration. Adding error handling and performance standards would elevate it to A+.

---

## Related Files

- Profile configuration: [profile-config.yml](profiles/android/profile-config.yml)
- Default standards location: `~/agent-os/profiles/default/standards/`
- Default workflows location: `~/agent-os/profiles/default/workflows/`
- Profile creation script: `~/agent-os/scripts/create-profile.sh`
- Agent OS base installation: `~/agent-os/`
- Configuration file: `~/agent-os/config.yml`

---

## Documentation Sources

### Getting Started
- [Agent OS Main Documentation](https://buildermethods.com/agent-os)
- [Installation Guide](https://buildermethods.com/agent-os/installation)
- [Configuration Options](https://buildermethods.com/agent-os/modes)
- [Updating Agent OS](https://buildermethods.com/agent-os/updating)
- [Tool Adaptability](https://buildermethods.com/agent-os/adaptability)
- [Version 2.0+ Features](https://buildermethods.com/agent-os/version-2)

### Workflow
- [Development Workflow Overview](https://buildermethods.com/agent-os/workflow)
- [Plan Product Command](https://buildermethods.com/agent-os/plan-product)
- [Shape Spec Command](https://buildermethods.com/agent-os/shape-spec)
- [Write Spec Command](https://buildermethods.com/agent-os/write-spec)
- [Create Tasks Command](https://buildermethods.com/agent-os/create-tasks)
- [Implement Tasks Command](https://buildermethods.com/agent-os/implement-tasks)
- [Orchestrate Tasks Command](https://buildermethods.com/agent-os/orchestrate-tasks)

### Core Concepts
- [Core Concepts Overview](https://buildermethods.com/agent-os/concepts)
- [3-Layer Context System](https://buildermethods.com/agent-os/3-layer-context)
- [Standards](https://buildermethods.com/agent-os/standards)
- [Skills (v2.1+)](https://buildermethods.com/agent-os/skills)
- [Profiles](https://buildermethods.com/agent-os/profiles)
- [Verification System](https://buildermethods.com/agent-os/verification)
- [Visuals](https://buildermethods.com/agent-os/visuals)
- [Workflows](https://buildermethods.com/agent-os/workflows)
