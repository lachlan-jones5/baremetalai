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
**Action**: Stages all uncommitted changes, adds them to the most recent commit (`HEAD`), and overwrites the old commit message.
**Instructions**:
1.  Run `git add .` to stage all new and modified files.
2.  Run `git commit --amend --no-edit` to add the staged changes to the `HEAD` commit without opening an editor.
3.  Analyze the *entire diff* of the now-amended `HEAD` commit by running `git show HEAD`.
4.  Based on this complete analysis, generate a new, comprehensive commit message, replacing the previous message. The previous commit message may provide useful context, but ensure the new message accurately reflects the current state of the commit.
5.  Overwrite the commit message of the amended commit by running `git commit --amend -m "newly generated message"`.

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
    * `Directory Structure`: Ensure the ASCII tree is accurate.
    * Any other sections that have become outdated due to new architectural decisions or implemented code.

