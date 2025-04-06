# AI Project Workflow Prompts

This document contains standardized prompts for AI interactions throughout your project lifecycle. These prompts are designed to maintain consistency across AI sessions and maximize the efficiency of your workflow.

## Prompt 1: Push Project to GitHub

Use this prompt at the end of a project to archive it to your GitHub repository.

```markdown
I want to archive our project to GitHub. Please help me organize all the code and files we've created into a well-formatted markdown document that I can easily reference and extract files from.

PROJECT NAME: [Name of your project]

PROJECT DESCRIPTION:
[Brief description of what the project does and its purpose]

PROGRAMMING LANGUAGE(S): [e.g., JavaScript, Python, PowerShell]

KEY FEATURES:
- [Feature 1]
- [Feature 2]
- [Feature 3]

LESSONS LEARNED:
- [What worked well]
- [Challenges faced]
- [Solutions found]
- [Technical discoveries]

USAGE EXAMPLES:
[Basic examples of how to use the code]

Please format your response as a clean markdown document with:
1. A project summary at the top
2. Each file clearly marked with a "## File: [filename]" heading
3. Code blocks using triple backticks with the appropriate language specified
4. Any additional documentation or instructions

For example, each file should follow this format:

## File: example.js
```javascript
// JavaScript code here
function example() {
  console.log("Hello world");
}
```

## File: styles.css
```css
/* CSS code here */
body {
  font-family: sans-serif;
}
```

I'll save this markdown document as my project reference and push it to my GitHub archive.
```

## Prompt 2: Continue Working on Existing Project

Use this prompt when you want to continue development on a previously archived project.

```markdown
I have an existing project documented in this reference file that I'd like to continue developing:

[Paste entire project_reference.md content]

Based on this reference, I'd like to add the following new features:
1. [New feature 1]
2. [New feature 2]
3. [New feature 3]

Please help me implement these features while maintaining compatibility with the existing codebase. For each feature, explain your approach and provide the complete implementation.
```

## Prompt 3: Share Knowledge Index When Struggling

Use this prompt when you're facing challenges and want to leverage your project history.

```markdown
I'm working on a project and having difficulty with [specific issue or challenge]. I have a collection of previous AI projects that might contain relevant solutions. Here's my project index with summaries of past work and lessons learned:

[Paste contents of ProjectIndex.md]

Based on this knowledge base:

1. Can you identify any relevant patterns or solutions from my previous projects that might help with my current challenge?
2. I'm currently trying to implement [specific feature/solution]. Which of my past projects has the most relevant example?
3. In project [specific project], we encountered [specific issue]. Can you explain how we solved it and how I might apply that approach to my current situation?
```

## Prompt 4: Get Implementation Advice Based on Past Projects

Use this prompt when starting a new implementation and wanting to leverage past experience.

```markdown
I need to implement [specific functionality] in [programming language]. Before I start, I'd like to leverage my existing codebase for insights.

Here's my project index:
[Paste contents of ProjectIndex.md]

Based on this index:
1. Which project(s) have similar functionality to what I'm trying to build?
2. What patterns or approaches would you recommend based on my previous work?
3. Are there any specific lessons learned from past projects that I should keep in mind?
4. Can you draft a high-level implementation plan that builds on my existing knowledge?
```

## Prompt 5: Debug With Historical Context

Use this prompt when debugging issues and wanting to reference past solutions.

```markdown
I'm debugging an issue in my code and would like to use my project history to help solve it.

Current Issue:
[Describe the bug or error you're experiencing]

Here's the relevant code:
```[language]
[Paste the problematic code]
```

Here's my project index that might contain similar issues I've solved before:
[Paste contents of ProjectIndex.md]

Can you:
1. Identify if I've encountered similar issues in past projects
2. Suggest debugging approaches based on my historical solutions
3. Provide a fix that's consistent with my coding patterns
```

[prompt6.md](https://github.com/user-attachments/files/19623789/prompt6.md)

I have an existing project that I want to convert into my standardized GitHub archival format. I'll provide you with the project files and information, and I need you to organize it into the markdown format used by my archival system.  

PROJECT NAME: [Name of your project]  

PROJECT DESCRIPTION:  
[Brief description of what the project does and its purpose]  

PROGRAMMING LANGUAGE(S): [e.g., JavaScript, Python, PowerShell]  

KEY FEATURES:  
- [Feature 1]  
- [Feature 2]  
- [Feature 3]  

LESSONS LEARNED:  
- [What worked well]  
- [Challenges faced]  
- [Solutions found]  
- [Technical discoveries]  

USAGE EXAMPLES:  
[Basic examples of how to use the code]  

HERE ARE THE PROJECT FILES:  

[File 1 Name: example.js]  
```javascript  
// Paste code here  
function example() {  
  console.log("Hello world");  
}  
