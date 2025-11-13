# Claude Code Configuration

This project uses specialized agents and output styles to optimize Claude Code for .NET web development with Blazor and JavaScript frontends.

## Output Style: Pragmatic Senior Developer

This project follows the **Pragmatic Senior Developer** approach defined in `claude/output-styles/pragmatic.md`:

- **Simple, Clean Solutions**: Straightforward code without unnecessary complexity
- **No Over-Engineering**: Build what's needed, nothing more
- **Stay in Scope**: Complete requested tasks without unsolicited refactoring
- **Ask, Don't Assume**: Clarify requirements before making assumptions
- **Minimal Initiative**: Only take initiative when necessary for task completion

## Project Context

**Technology Stack**:
- Backend: C# / .NET 6+, ASP.NET Core Web API, Entity Framework Core
- Frontend: Blazor (Server & WebAssembly), JavaScript/TypeScript
- Patterns: Repository Pattern, Dependency Injection, SOLID Principles

## Available Specialized Agents

This project includes 5 specialized agents in the `agents/` directory. Use these by referencing them in your prompts to get expert-level assistance for specific tasks.

### 1. Requirements Gatherer (`agents/requirements-gatherer.md`)
**Use when**: Starting new features, unclear requirements, or planning complex implementations

**Example prompt**:
```
Using agents/requirements-gatherer.md, help me document requirements for a new user dashboard feature
```

**What it does**:
- Analyzes existing codebase patterns
- Asks clarifying questions about functional and technical requirements
- Documents comprehensive specifications
- Identifies integration points and dependencies
- Creates acceptance criteria

---

### 2. Code Writer (`agents/code-writer.md`)
**Use when**: Implementing new features, creating components, or writing API endpoints

**Example prompt**:
```
Using agents/code-writer.md, implement a new order processing API endpoint with validation and error handling
```

**What it does**:
- Writes production-ready .NET, Blazor, and JavaScript code
- Follows SOLID principles and best practices
- Implements proper async/await patterns
- Includes error handling and logging
- Uses appropriate design patterns

---

### 3. Code Reviewer (`agents/code-reviewer.md`)
**Use when**: Before committing code, during PR reviews, or for quality assurance

**Example prompt**:
```
Using agents/code-reviewer.md, review the OrderController.cs file and provide feedback on code quality and security
```

**What it does**:
- Performs thorough code quality reviews
- Checks SOLID principles adherence
- Reviews security vulnerabilities (OWASP Top 10)
- Identifies performance issues
- Validates best practices for .NET/Blazor/JavaScript
- Provides categorized feedback (Critical/Warning/Suggestion)

---

### 4. Refactoring Agent (`agents/refactoring-agent.md`)
**Use when**: Improving code quality, reducing technical debt, or modernizing legacy code

**Example prompt**:
```
Using agents/refactoring-agent.md, refactor the UserService to follow the repository pattern
```

**What it does**:
- Identifies code smells and anti-patterns
- Suggests architectural improvements
- Preserves functionality while improving code
- Modernizes code to latest C# features
- Provides before/after comparisons

---

### 5. Security Analyzer (`agents/security-analyzer.md`)
**Use when**: Security audits, before production deployment, or reviewing authentication/authorization

**Example prompt**:
```
Using agents/security-analyzer.md, perform a security analysis on the authentication and authorization implementation
```

**What it does**:
- Identifies OWASP Top 10 vulnerabilities
- Analyzes .NET-specific security issues
- Reviews authentication and authorization
- Checks for injection attacks, XSS, CSRF
- Provides prioritized remediation guidance

---

## Common Workflows

### Full Feature Development Cycle
```
1. Using agents/requirements-gatherer.md, document requirements for [feature name]
2. Using agents/code-writer.md, implement [feature] based on the requirements
3. Using agents/code-reviewer.md, review the code we just wrote
4. Using agents/security-analyzer.md, analyze security of the new feature
```

### Code Quality Improvement
```
1. Using agents/code-reviewer.md, analyze [filename] and identify issues
2. Using agents/refactoring-agent.md, refactor [filename] to address the issues
3. Using agents/code-reviewer.md, verify the improvements
```

### Security Audit
```
1. Using agents/security-analyzer.md, perform security audit focusing on authentication, API endpoints, and data validation
2. Using agents/code-writer.md, implement the security fixes identified
3. Using agents/security-analyzer.md, verify the fixes are properly implemented
```

---

## Best Practices

### When Working with Claude Code on This Project

1. **Be Specific**: Reference specific files, components, or areas of concern
   ```
   ❌ "Review my code"
   ✅ "Using agents/code-reviewer.md, review src/Controllers/OrderController.cs focusing on error handling"
   ```

2. **Provide Context**: Mention constraints, existing patterns, or project-specific requirements
   ```
   ✅ "We use the repository pattern and Entity Framework Core. Write a new CustomerService following these patterns."
   ```

3. **Chain Agents**: Use multiple agents in sequence for comprehensive workflows
   ```
   ✅ "First gather requirements, then implement, then review"
   ```

4. **Reference the Output Style**: The pragmatic approach means:
   - Solutions will be straightforward and focused
   - No unsolicited refactoring of unrelated code
   - Questions will be asked when requirements are unclear
   - Code will follow existing project conventions

---

## Project Standards

### Code Quality
- Follow SOLID principles
- Use async/await for I/O operations
- Implement proper dependency injection
- Include XML documentation for public APIs
- Use data annotations for validation

### Security
- Validate all user input
- Use parameterized queries (EF Core)
- Implement proper authentication/authorization
- Follow OWASP guidelines
- Never store secrets in code

---

## How to Use This Configuration

### Quick Start
Simply reference an agent in your prompt:
```
Using agents/code-writer.md, create a new ProductService
```

### Multiple Agents
Chain multiple agents for complex tasks:
```
1. Using agents/requirements-gatherer.md, analyze what's needed
2. Using agents/code-writer.md, implement the solution
3. Using agents/code-reviewer.md, review the implementation
```

### Without Agents
For simple tasks, just ask directly. The pragmatic output style will be applied automatically.

---

## Customization

### Modifying Agents
Edit files in `agents/` to:
- Add company-specific standards
- Include internal documentation references
- Add project-specific checks
- Adjust review criteria

### Creating New Agents
Use existing agents as templates to create new ones for:
- Database design
- Performance optimization
- API documentation
- DevOps/deployment

### Adjusting Output Style
Modify `claude/output-styles/pragmatic.md` to:
- Change communication style
- Adjust level of initiative
- Add project-specific guidelines

---

## Support

- **Agent Documentation**: See `agents/README.md` for detailed agent information
- **Claude Code Help**: Run `/help` or visit https://docs.claude.com
- **.NET Best Practices**: Reference Microsoft documentation
- **Security Guidelines**: Reference OWASP guidelines

---

## Quick Reference

| Task | Agent | Command |
|------|-------|---------|
| Gather requirements | requirements-gatherer | `Using agents/requirements-gatherer.md, ...` |
| Write new code | code-writer | `Using agents/code-writer.md, ...` |
| Review code | code-reviewer | `Using agents/code-reviewer.md, ...` |
| Refactor code | refactoring-agent | `Using agents/refactoring-agent.md, ...` |
| Security audit | security-analyzer | `Using agents/security-analyzer.md, ...` |

---

**Version**: 1.0
**Last Updated**: 2025-11-13
**Optimized For**: .NET 6+, Blazor, ASP.NET Core, Entity Framework Core
