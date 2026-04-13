# d-eng-team
An AI Agent plugin to mange software engineering team AI workflow


# Initiative

Manage an AI engineering team with:

- Document as memory.
- Different roles with separate sessions, hence separated contexts and memories as well.
- Agent with high turn over rate.


# Ideology

- Skill **onboarding**: gets the repo ready for this workflow, ensures that the `AGENTS.md`/`CLAUDE.md` contains necessary intruction for this workflow
- Document structure:
  * Three level of documents:
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
  * An intruction / indexing system for agent to reference. A new agent without any context and memory, can follow this trail to understand the project and start to work on specific task with minimum effort/context.
    1. When a new agent get a problem statement, it read the **Highlevel design** to understand the project and which module to work with.
    2. Read the **Module design** to understand further and work on the solution
    3. For cross module dependencies and interactions, refer to target module's **Detail design**
    4. Update **Module design** and **Detail design** of the module it's working on
    5. If modification / support from other module is required, trigger **architect** to coordinate.
  * A knowledge base with  index. Agent can get knowledge on demand. For example, title: Bug in foo() function in base library, link to a document that describe this bug in detail and the solution to avoid it. Agent only keep the title in the context, only during troubleshooting or using this function will the agent load the detail into the context.


# Roles

- PO
  * **Input**: user input
  * **Output**: user stories, requirements, acceptance criterias.
- Architect
  * **Scope**: project level
  * **Input**: requirements, user stories
  * **Output**: Highlevel design, instructions to **Developer** and **QA**
- Developer
  * **Scope**: module level
- QA
- Librarian





# Notes

- Developer should focus on specific module. With full knowledge of the module, knowledge about the interfaces related to this module, the shape of the project and the high level design instruction given by architect.
- Architect with full knowledge of the system design, full knowledge about the module boundaries, breaks down the requirement and design into module level.
