# Claude Code Sub-Agents for .NET Development

This directory contains specialized sub-agent definitions for Claude Code, tailored for senior .NET developers working with web applications using JavaScript and Blazor frontends.

## Available Agents

### 1. Requirements Gatherer (`requirements-gatherer.md`)
**Purpose**: Analyze, document, and clarify requirements for new features or changes.

**When to use**:
- Starting a new feature
- Unclear or incomplete requirements
- Need to document a feature specification
- Planning a complex implementation

**Example usage**:
```
I need help gathering requirements for a new user dashboard feature
```

### 2. Code Writer (`code-writer.md`)
**Purpose**: Write production-ready .NET, Blazor, and JavaScript code following best practices.

**When to use**:
- Implementing new features
- Creating new components or services
- Writing API endpoints
- Building Blazor components

**Example usage**:
```
Write a new order processing API endpoint with validation and error handling
```

### 3. Code Reviewer (`code-reviewer.md`)
**Purpose**: Perform thorough code reviews with actionable feedback.

**When to use**:
- Before committing code
- During pull request review
- Quality assurance checks
- Learning best practices

**Example usage**:
```
Review the OrderController.cs file and provide feedback on code quality and security
```

### 4. Refactoring Agent (`refactoring-agent.md`)
**Purpose**: Improve code quality, reduce technical debt, and modernize code.

**When to use**:
- Code smells identified
- Technical debt reduction
- Performance improvements
- Modernizing legacy code
- Improving maintainability

**Example usage**:
```
Refactor the UserService to follow the repository pattern and improve testability
```

### 5. Security Analyzer (`security-analyzer.md`)
**Purpose**: Identify security vulnerabilities and provide remediation guidance.

**When to use**:
- Security audits
- Before production deployment
- After adding authentication/authorization
- Reviewing API security
- Compliance requirements

**Example usage**:
```
Perform a security analysis on the authentication and authorization implementation
```

## How to Use These Agents

### Method 1: Direct Reference in Prompts
Copy and paste the content from the agent file into your prompt:

```
[Copy content from agents/code-reviewer.md]

Please review my OrderController.cs file.
```

### Method 2: Reference by File Path
Ask Claude Code to read and apply the agent definition:

```
Read the agent definition from agents/code-reviewer.md and use it to review my OrderController.cs file.
```

### Method 3: Task Tool with Context
Use the Task tool and reference the agent file for context:

```
Using the guidelines in agents/security-analyzer.md, perform a security analysis of my web API.
```

## Agent Workflow Examples

### Example 1: Full Feature Development Cycle

**Step 1: Requirements**
```
Using agents/requirements-gatherer.md, help me document requirements for a new invoice management feature.
```

**Step 2: Implementation**
```
Using agents/code-writer.md, implement the invoice management feature based on the requirements we just documented.
```

**Step 3: Code Review**
```
Using agents/code-reviewer.md, review the invoice management code we just wrote.
```

**Step 4: Security Check**
```
Using agents/security-analyzer.md, analyze the security of the new invoice feature.
```

### Example 2: Legacy Code Improvement

**Step 1: Analysis**
```
Using agents/code-reviewer.md, analyze the legacy OrderService.cs and identify issues.
```

**Step 2: Refactoring**
```
Using agents/refactoring-agent.md, refactor the OrderService to modern .NET standards.
```

**Step 3: Security Review**
```
Using agents/security-analyzer.md, check if the refactored code has any security issues.
```

### Example 3: Security Audit

**Step 1: Comprehensive Analysis**
```
Using agents/security-analyzer.md, perform a complete security audit of the application focusing on:
1. Authentication/Authorization
2. API endpoints
3. Data validation
4. OWASP Top 10
```

**Step 2: Remediation**
```
Using agents/code-writer.md, implement the security fixes identified in the audit.
```

**Step 3: Verification**
```
Using agents/code-reviewer.md, verify that the security fixes are properly implemented.
```

## Best Practices

### 1. Choose the Right Agent
- Use **Requirements Gatherer** when things are unclear
- Use **Code Writer** for implementation tasks
- Use **Code Reviewer** for quality assurance
- Use **Refactoring Agent** for improvements
- Use **Security Analyzer** for security concerns

### 2. Combine Agents
Agents work well together in sequence:
- Requirements → Code Writing → Review → Security Analysis

### 3. Be Specific
When invoking an agent, be specific about:
- What files to analyze
- What aspect to focus on
- What output you expect

### 4. Iterative Approach
- Start with one agent
- Review the output
- Use another agent to build on the results
- Iterate as needed

### 5. Context Matters
Provide agents with:
- Relevant file paths
- Project constraints
- Existing patterns in your codebase
- Specific concerns or requirements

## Customization

These agents are templates. You can:

### Modify Existing Agents
Edit the .md files to:
- Add company-specific standards
- Include custom coding guidelines
- Reference internal documentation
- Add project-specific checks

### Create New Agents
Use the existing agents as templates to create new ones for:
- Database design
- API documentation
- Performance optimization
- Testing strategies
- DevOps/deployment

## Tips for Effective Use

### 1. Start Broad, Then Narrow
```
Bad: "Review my code"
Good: "Using agents/code-reviewer.md, review OrderController.cs focusing on error handling and validation"
```

### 2. Provide Context
```
Bad: "Write a service"
Good: "Using agents/code-writer.md, write an OrderService that follows our repository pattern and uses Entity Framework Core"
```

### 3. Ask for Specific Output
```
Bad: "Analyze security"
Good: "Using agents/security-analyzer.md, analyze authentication in UserController.cs and provide a report with prioritized findings"
```

### 4. Chain Agents
```
First: "Using agents/requirements-gatherer.md, document requirements for user profile editing"
Then: "Using agents/code-writer.md, implement the user profile editing based on those requirements"
Finally: "Using agents/code-reviewer.md, review the implementation"
```

## Integration with Development Workflow

### During Development
1. **Requirements Gatherer** - Before starting work
2. **Code Writer** - During implementation
3. **Code Reviewer** - Before committing

### During Code Review
1. **Code Reviewer** - First pass review
2. **Security Analyzer** - Security-focused review
3. **Refactoring Agent** - Identify improvements

### During Maintenance
1. **Code Reviewer** - Assess current state
2. **Refactoring Agent** - Plan improvements
3. **Code Writer** - Implement changes
4. **Security Analyzer** - Verify security

## Technical Stack Context

These agents are optimized for:

### Backend
- C# / .NET 6+
- ASP.NET Core Web API
- Entity Framework Core
- Dependency Injection
- Repository Pattern
- SOLID Principles

### Frontend
- Blazor Server
- Blazor WebAssembly
- JavaScript/TypeScript
- Razor Components
- JavaScript Interop

### Testing
- xUnit / NUnit / MSTest
- Moq
- bUnit (Blazor testing)
- Integration Testing

### Security
- ASP.NET Core Identity
- JWT Authentication
- OWASP Top 10
- GDPR Compliance

## Troubleshooting

### Agent Not Responding as Expected
- Ensure you're providing enough context
- Reference specific files or code sections
- Be explicit about what aspect to focus on

### Too Generic Output
- Be more specific in your request
- Provide code examples
- Reference the specific section of the agent you want applied

### Need Different Perspective
- Try combining multiple agents
- Use agents in different order
- Customize the agent definitions for your needs

## Contributing

To improve these agents:

1. Identify gaps or areas for improvement
2. Edit the relevant .md file
3. Test the changes with real scenarios
4. Document any new patterns or guidelines

## Support

For issues or questions about:
- **Claude Code**: Check Claude Code documentation
- **These Agents**: Modify the agents to fit your needs
- **.NET Best Practices**: Reference Microsoft documentation
- **Security**: Reference OWASP guidelines

## Version History

- **v1.0** - Initial release with 5 core agents for .NET development
  - Requirements Gatherer
  - Code Writer
  - Code Reviewer
  - Refactoring Agent
  - Security Analyzer

## License

These agent definitions are tools for use with Claude Code. Modify and adapt them to fit your team's needs and coding standards.
