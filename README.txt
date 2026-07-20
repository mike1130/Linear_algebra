====================================================================
GIT & GITHUB QUICK REFERENCE
Setup, daily workflow, branches, merges, .gitignore, troubleshooting
====================================================================

--------------------------------------------------------------------
1. CHECK IF GIT IS INSTALLED AND CONFIGURED
--------------------------------------------------------------------
git --version
    -> Prints a version number if installed.
    -> "command not found" = install from https://git-scm.com

git config --global user.name
git config --global user.email
    -> If empty, configure (use your GitHub email):
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

--------------------------------------------------------------------
2. FIRST-TIME SETUP: LINK A LOCAL FOLDER TO A NEW GITHUB REPO
--------------------------------------------------------------------
a) On github.com: + (top right) -> New repository
   -> Do NOT check "Add a README" (keeps first push simple)

b) In your local folder:
cd path\to\your\folder
git init
git add .
git commit -m "Initial commit"
git remote add origin git@github.com:USERNAME/REPO.git
    (or HTTPS: https://github.com/USERNAME/REPO.git)
git branch -M main
git push -u origin main

    -> The -u flag sets up tracking: future pushes only need
       "git push"

--------------------------------------------------------------------
3. UPDATE GITHUB (the daily loop)
--------------------------------------------------------------------
git status                      # see what changed
git add .                       # stage all changes
git commit -m "describe what changed"
git push                        # upload to GitHub

Tip: run "git status" before and after each step - it always
tells you what state you're in and suggests the next command.

--------------------------------------------------------------------
4. THE MENTAL MODEL: WHAT "main" ACTUALLY IS
--------------------------------------------------------------------
Every "git commit" creates a snapshot (a dot). "main" is just a
label pointing at the latest one - it moves forward with each
commit. The chain is never overwritten, only extended, which is
why any old version can be restored.

                                              [main]
                                                |
                                                v
    (a1f3) ----> (b7e2) ----> (c9d8) ----> (e4a1)
   Initial      Add notes    Fix ch.2     Add exercises
   commit

Each dot has an ID (the hash shown in "git log").

--------------------------------------------------------------------
5. BRANCHES: PARALLEL TIMELINES
--------------------------------------------------------------------
"git checkout -b experiment" creates a second label. New commits
grow a separate line while main stays untouched - try something
risky without breaking the working version.

                     (f2b4) ----> (d6c3)   [experiment]
                    /
    (a1f3) --> (b7e2) --> (c9d8)           [main]

Commands:
git checkout -b experiment   # create branch and switch to it
    ...edit files, commit as usual...
git checkout main            # switch back - your files
                             # instantly revert to main's version

NOTE: switching branches rewrites the files in your folder to
match that branch's latest commit. Nothing is lost - it's just
showing a different snapshot.

--------------------------------------------------------------------
6. MERGING: BRINGING THE BRANCH BACK
--------------------------------------------------------------------
"git merge" combines both histories into a new commit that has
TWO parents:

                (f2b4) ----> (d6c3)
               /                   \
    (b7e2) --> (c9d8) ------------> (MERGE)   [main]

Commands:
git checkout main
git merge experiment         # pull experiment's changes into main
git branch -d experiment     # delete the label
                             # (the commits stay in history)

MERGE CONFLICTS: if both branches edited the same lines of the
same file, git marks the section in the file with <<<<<<< and
>>>>>>> markers. Edit the file to keep what you want, then
git add + git commit. It's just text editing.

--------------------------------------------------------------------
7. OTHER COMMANDS WORTH KNOWING EARLY
--------------------------------------------------------------------
git log --oneline --graph    # see your commit chain drawn in
                             # ASCII, like the diagrams above
git diff                     # what changed since last commit
git restore file.txt         # discard uncommitted changes
git revert a1f3              # undo a commit by creating a new
                             # "opposite" commit
git pull                     # fetch + merge changes from GitHub
                             # into your local copy

--------------------------------------------------------------------
8. THE .GITIGNORE FILE
--------------------------------------------------------------------
A plain text file named exactly ".gitignore" in the repo root.
Each line is a pattern; matching files become invisible to
"git add ." and "git status".

# Comments start with #

# Ignore a specific file
secrets.txt

# Ignore every file with an extension, anywhere in the repo
*.log
*.tmp
*.pyc

# Ignore an entire folder (trailing slash = folder)
datasets/
venv/
__pycache__/
node_modules/

# Ignore a folder only at the root (leading slash = anchor)
/build/

# Ignore everything in a folder EXCEPT one file (! = exception)
results/*
!results/summary.txt

# Ignore OS junk
.DS_Store
Thumbs.db

Rules in one breath: * is a wildcard, trailing / means directory,
leading / means "only at the root", ! re-includes something,
# is a comment.

GOTCHAS:
1. .gitignore only affects UNTRACKED files. If a file was already
   committed, git keeps tracking it. Fix:
       git rm --cached file.log
   then commit - the file stays on disk but leaves the repo.
2. Commit the .gitignore itself (git add .gitignore) so the
   rules travel with the repo.

Starter for a Python/Jupyter project:
__pycache__/
venv/
.ipynb_checkpoints/
*.pyc
.DS_Store

--------------------------------------------------------------------
9. POSSIBLE PROBLEMS AND FIXES
--------------------------------------------------------------------

PROBLEM: "failed to push some refs"
CAUSE 1: No commits yet ("git branch" shows nothing).
FIX:
    git add .
    git commit -m "Initial commit"
    git branch -M main
    git push -u origin main

CAUSE 2: Repo on GitHub was created with a README, so remote
has a commit your local folder doesn't.
FIX:
    git pull origin main --allow-unrelated-histories
    git push -u origin main

CAUSE 3: Branch name mismatch (local "master" vs GitHub "main").
FIX:
    git branch -M main
    git push -u origin main

--------------------------------------------------------------------
PROBLEM: "The authenticity of host 'github.com' can't be
established" (fingerprint prompt)
FIX: Normal on first SSH connection. Type "yes" and press Enter.

--------------------------------------------------------------------
PROBLEM: "Permission denied (publickey)"
CAUSE: No SSH key registered with GitHub, or git is using a
different SSH program than the one holding your key.

FIX A - Create and register an SSH key:
    ssh-keygen -t ed25519 -C "your-github-email@example.com"
        (press Enter at each prompt)
    type %USERPROFILE%\.ssh\id_ed25519.pub
        (Windows CMD; on Git Bash/Linux/Mac use:
         cat ~/.ssh/id_ed25519.pub)
    Copy the output -> GitHub -> Settings -> SSH and GPG keys
    -> New SSH key -> paste -> Add SSH key
    Test:
    ssh -T git@github.com
        -> "Hi USERNAME! You've successfully authenticated" = OK

FIX B - "ssh -T" works but "git push" still denied:
CAUSE: Windows has two SSH programs (System32 OpenSSH and
Git's bundled SSH); your key works with one, git uses the other.
    git config --global core.sshCommand "C:/Windows/System32/OpenSSH/ssh.exe"
    git push -u origin main

FIX C - Skip SSH entirely, use HTTPS (simplest for beginners):
    git remote set-url origin https://github.com/USERNAME/REPO.git
    git push -u origin main
        -> Sign in via browser/token once; Windows saves it.

--------------------------------------------------------------------
PROBLEM: "'cat' n'est pas reconnu..." (command not found in CMD)
CAUSE: cat and ~ are Linux/Mac syntax; Windows CMD differs.
FIX: Use "type" and %USERPROFILE% instead:
    type %USERPROFILE%\.ssh\id_ed25519.pub
BETTER: Use Git Bash (installed with git, in Start menu) -
it accepts Linux-style commands used in most tutorials.

--------------------------------------------------------------------
10. SIGNS OF A SUCCESSFUL PUSH
--------------------------------------------------------------------
Output like:
    Writing objects: 100% ...
    * [new branch] main -> main
    branch 'main' set up to track 'origin/main'
-> Refresh the repo page on github.com to see your files.
-> If files are missing, run "git status": untracked files were
   never committed. Add, commit, and push them.

====================================================================
Mental model in one sentence: your folder is the working copy,
"commit" saves versioned snapshots locally, and "push"/"pull"
sync those snapshots with GitHub.
====================================================================