## Custom Instructions

### Git Workflow

#### @git/commit
**Action**: Stages all uncommitted changes and creates a new commit.
**Instructions**:
1.  Run `git add .` to stage all new and modified files.
2.  Analyze the staged changes by running `git diff --staged`.
3.  Based on the analysis, generate a concise and descriptive commit message following the conventional commit format (e.g., `feat(runtime): implement SRAM read/write logic`).
4.  Execute the commit using the generated message: `git commit -m "..."`.

#### @git/amend
**Action**: Stages all uncommitted changes, adds them to the most recent commit, then updates the commit message based on the complete diff.
**Instructions**:
1.  Run `git add .` to stage all new and modified files.
2.  Analyze the *entire diff* that will make up the amended commit by running `git diff HEAD~1`.
3.  Based on this complete analysis, generate a new, comprehensive commit message. The previous commit message must be ignored; the new message should be based solely on the current state of the commit.
4.  Amend the most recent commit, updating both its contents and message, by running `git commit --amend -m "newly generated message"`.

### Project Management

#### @git/readme
**Action**: Analyzes the entire repository and updates the `README.md` file to reflect the current state of the project.
**Instructions**:
1.  Perform a comprehensive analysis of the entire repository. This includes:
    * The current file and directory structure.
    * The contents of all source files, configuration files, and documentation.
    * The project's actual implementation status versus what's described in the README.
    * Any build systems, dependencies, or tooling present.
    * TODO comments or issue markers in the code.
2.  Based on this analysis, update the `README.md` file to accurately reflect the current state:
    * Add or update a `Repository Structure` section with an ASCII tree representation of the directory structure.
    * Update features and implementation details to match reality.
    * Ensure all referenced files, directories, and documentation actually exist.
    * Update technology stack and dependencies based on actual files present.
    * Keep the existing tone and style of the README while ensuring accuracy.
3.  Do NOT use git commands to analyze changes - instead manually review all files and directories to understand the current state.

#### @copilot/update
**Action**: Analyzes the entire repository and updates this `copilot-instructions.md` file to reflect the current state of the project.
**Instructions**:
1.  Perform a comprehensive analysis of the entire repository. This includes:
    * The current file and directory structure.
    * The contents of `CMakeLists.txt` files to understand build targets.
    * The public interfaces of C++ header files.
    * TODO comments or issue markers in the code.
2.  Based on this analysis, update the following sections of this document:
    * `Directory Structure`: Ensure the ASCII tree is accurate. If one does not exist, generate a new ASCII tree representation.
    * Any other sections that have become outdated due to new architectural decisions or implemented code.

