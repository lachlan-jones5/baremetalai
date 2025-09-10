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
2.  Analyze the *entire diff* that will make up the amended commit by running `git diff --cached`.
3.  Based on this complete analysis, generate a new, comprehensive commit message. The previous commit message may provide useful context, but ensure the new message accurately reflects the current state of the commit.
4.  Amend the most recent commit, updating both its contents and message, by running `git commit --amend -m "newly generated message"`.

### Project Management

#### @copilot/update
**Action**: Analyzes the entire repository and updates this `copilot-instructions.md` file to reflect the current state of the project.
**Instructions**:
1.  Perform a comprehensive analysis of the entire repository. This includes:
    * The current file and directory structure.
    * The contents of `CMakeLists.txt` files to understand build targets.
    * The public interfaces of C++ header files.
    * TODO comments or issue markers in the code.
2.  Based on this analysis, update the following sections of this document:
    * `Directory Structure`: Ensure the ASCII tree is accurate. If one does not exist, create one yourself.
    * Any other sections that have become outdated due to new architectural decisions or implemented code.

