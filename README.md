# d-eng-team
An AI Agent plugin to manage software engineering team AI workflow

# Glossary

- **the plugin**: this plugin
- **the project** / **target project**: the repo/project that installs this plugin
- **the user**: the human who is working on the **project**
- **the workflow**: the way that this plugin works.

# Initiative

Manage an AI engineering team for the target project, with:

- Document as memory.
- Different roles with separate sessions, hence separated contexts and memories as well.
- Agents with a high turnover rate, don't just rely on the session / context memory.


# Ideology

- Skill **onboarding**: gets the repo ready for this workflow, ensures that the `AGENTS.md`/`CLAUDE.md` contains necessary instruction for this workflow
- Document structure:
  * Three levels of design documents:
    - **Highlevel design**:
      * Key business flow
      * Modules boundaries and relationships
    - **Module design**:
      * Module logical design
      * Module _internal_ usage, for who works on this module to reference.
    - **Detail design**:
      * API spec
      * detail flow chart
      * Exceptional handling
      * Module _external_ usage, for those who works with this module to reference.
  * An instruction / indexing system for agent to reference. A new agent without any context and memory, can follow this trail to understand the project and start to work on specific task with minimum effort/context.
    1. When a new agent gets a problem statement, it reads the **Highlevel design** to understand the project and which module to work with.
    2. Read the **Module design** to understand further and work on the solution
    3. For cross module dependencies and interactions, refer to target module's **Detail design**
    4. Update **Module design** and **Detail design** of the module it's working on
    5. If modification / support from other module is required, trigger **architect** to coordinate.
  * A knowledge base with index. Agent can get knowledge on demand. For example, title: Bug in foo() function in base library, link to a document that describe this bug in detail and the solution to avoid it. Agent only keep the title in the context, only during troubleshooting or using this function will the agent load the detail into the context.


# Documents



| Document            | Owner            | Descriptions                                                                                                  |
|---------------------|------------------|---------------------------------------------------------------------------------------------------------------|
| Initiative          | Business partner | Target and end-goal of the project. A north star of the project.                                              |
| Use cases           | Business analyst | - Use cases for business scenarios<br>- Includes acceptance criteria<br>- Includes the exceptional scenarios |
| Detail requirements | Business analyst | Supplementary document of **Use cases**. Provide detail requirement of Use cases                              |
| ADR                 | Architect        | - Architecture decision record, record every key architecture decisions                                       |
| Highlevel design    | Architect        | - Highlevel design of the project<br>- Focus on highlevel flow and the boundary of modules                     |
| Module design       | Engineer         | - Detail logic of the module                                                                                  |
| Detail design       | Engineer         | - Interface contract of the module                                                                            |
| Test cases          | QA               | - Test case<br>- User test cases and instructions                                                             |
| Manuals             | Architect        | - `Usage.md`<br>- `User Manuals.md`                                                                           |
| Index               | Librarian        | - index of all documents<br>- provide index for agent to reference                                            |


# Roles

- Business analyst
  * **Input**: user input
  * **Output**: user stories, requirements, acceptance criteria.
- Architect
  * **Scope**: project level
  * **Input**: requirements, user stories
  * **Output**: Highlevel design, instructions to **Developer** and **QA**
- Developer
  * **Scope**: module level
- QA
- Librarian
  * Organize the document base


| role             | is orchestrator | counterparts                    | Document scope                                           |
|------------------|-----------------|---------------------------------|----------------------------------------------------------|
| Business partner | No              | Business analyst                | Initiative                                               |
| Business analyst | No              | Architect                       | Highlevel requirement<br>Detail requirement<br>Use cases |
| Architect        | Yes             | Everyone                        | Highlevel design                                         |
| Engineer         | No              | Architect, QA, Business analyst | Module design<br>Detail design                           |
| QA               | No              | Engineer, Business analyst      | Module design<br>Detail design<br>Test cases             |
| Librarian        | Yes             | Everyone                        | All                                                      |


# Orchestration

**Orchestrator**s can command other roles which might be running in other session/agents. There should be a contract between the commander and executor.

## Request format
A prompt
```
From: <from agent name>
To: <target agent name>
---
<request prompt body>
```

## Response format
```
From: <from agent name>
To: <to agent name>
Status: <status of the prompt>
---
<respond body>
```

- **Status**:
  * **Done**: job done. The response body is the result of the work done.
  * **Stuck**: job stuck by insufficient information or issues cannot be fixed within the scope of current agent. The requestor should clarify / coordinate and fix the issue / escalate to user. Once issue fixed, the requestor should request the target to continue. In this status, the response should be the reason of the blockage.


The plugin should provide skills for both requestor and the target.
- For requestor: request format, execute prompt on different agent session
- For target: response format

# Plugin structure

- `document-structure.md`: Define the document structure for the project that use this plugin. `/onboarding` skill will install reference to this document to the project's owned agent instruction file, such as `CLAUDE.md`/`AGENTS.md`.
- `skills/`
  * `onboarding/SKILL.md`: A skill to help the project to initialize with this plugin. Including the agent instructions injection.
- `agents/`
  * `ba`: Business analyst, focus on understanding user's requirements. Output user stories, requirements, acceptance criteria.
  * `architect`: The core of this workflow. It takes in requirements, user stories, discuss with the user, output the highlevel design.




# Notes

- Developer should focus on specific module. With full knowledge of the module, knowledge about the interfaces related to this module, the shape of the project and the high level design instruction given by architect.
- Architect with full knowledge of the system design, full knowledge about the module boundaries, breaks down the requirement and design into module level.
