# Lab: Coding Agents

In this lab you will create a simple coding agent based on `llm` or `dic`.

<img src=img/xkcd.png width=300px />

Your agent will rely on git *patch files*.
Patch files are core to the Linux and Python development process,
and you'll notice that Linus and Guido both helped write portions of this lab.

<img src=img/contrib.png width=200px />

You'll need a partner for Part 2 to practice the [Linux patchfile contribution process](https://docs.kernel.org/process/applying-patches.html),
which is slightly more technical than the github pull request.
The rest of this lab can be completed alone
(but you are of course encouraged to collaborate with other biologicals).

## Part 0: setup

Clone the repo.

```
$ git clone https://github.com/mikeizbicki/lab-coding-agents
$ cd lab-coding-agents
```

Observe that this repo contains a *submodule* `lab-cat` inside of it.
(A submodule is a git repo inside of another git repo.)
By default, `git clone` does not download submodules when cloning.
Observe that the `lab-cat` folder is empty:
```
$ ls lab-cat
```

You can download the contents with the `submodule update` git command:
```
$ git submodule update --init --recursive
$ ls lab-cat
```

Inside of this repo is a basic data structures assignment for understanding the difference between constant and linear memory algorithms.
The idea is that the existing `cat.py` file used $O(n)$ memory and so cannot work with large files,
and the assignment is to change it to an $O(1)$ memory implementation
(just like the built-in `cat` program).

In this lab, you will solve this `lab-cat` submodule in 3 ways with different levels of automation.

## Part 1: pseudo-manually fixing

Inside the `lab-cat` submodule create a new branch `pseudomanual`:
```
$ cd lab-cat
$ git checkout -b pseudomanual
```

Soon we will see how to use qwen to solve the lab for us.
But first, let's practice with the `files-to-prompt` command:
```
$ files-to-prompt cat.py
<...>
$ files-to-prompt .
<...>
```
You should observe that `files-to-prompt` is similar to the built-in `cat` but with two differences:
1. it prints the name of the file before printing the contents
2. when passed a directory, it prints all non-hidden / non-binary files in the directory
That makes it particularly good for feeding files into llms.

We can get qwen to write the corrected python code for us by running
```
$ qwen <<EOF
$(files-to-prompt .)

Fix the python code.
EOF
```

> **NOTE:**
> The command above will likely give you an error about the llm refusing to obey your instructions.
> This is due to a *prompt injection attack* in the README file where I overwrite your instructions of `Fix the python code` with my own instructions.
> LLMs have no built-in way of identifying which text is "instructions",
> and which text is just "background knowledge".
>
> You can get your `qwen` command to work by either:
> 1. modifying README to remove the prompt injection, or
> 2. actually providing the testcases by modifying the `files-to-prompt` command to explicitly include the `.github` folder (recall that hidden files are ignored by default in `files-to-prompt`).
>
> A command like the following will add the test cases and so should work:
> ```
> $ qwen <<EOF
> $(files-to-prompt . .github)
> 
> Fix the python code.
> EOF
> ```
> Recall that when you are asking your own questions to llms,
> you will always get much better responses if you include the test cases in the context.

Copy/paste the output of `qwen` to vim in order to fix the `cat.py` file.
Then add/commit your new code.
```
$ git add cat.py
$ git commit -m 'fixed by qwen'
```

## Part 2: git diffs

A git diff shows the difference between your current code and a different branch/commit.
Run the command:
```
$ git diff master
```
To show the difference between your `pseudomanual` branch and `master`.
The exact output will be different for everyone because llms are nondeterministic.
But the output should look something like
```
diff --git a/cat.py b/cat.py
index f879933..19d6cf5 100644
--- a/cat.py
+++ b/cat.py
@@ -4,8 +4,8 @@ This program prints stdin to the screen.
 import sys
 
 def cat(file):
-    data = file.read()
-    sys.stdout.buffer.write(data)
+    while chunk := file.read(8192):
+        sys.stdout.buffer.write(chunk)
 
 if __name__ == "__main__":
     if len(sys.argv) > 1:
```
Observe that the lines starting with `-` are supposed to be deleted and the lines starting with `+` are supposed to be added to convert the `master` branch into your `pseudomanual` branch.

When these diffs are stored in files, they are commonly called *patch files*.
And they can be used to directly change your code.
Sending raw patch files to other people was the original way to submit "pull requests" to other people before github.com was founded.
Many open source projects like the Linux kernel still use an email-based workflow using patchfiles and no website.

In the rest of this section, you are going to walk through this manual pull requestprocedure to learn how patch files work.
You'll need a partner for these steps.

**SETUP STEPS:**

1. Create a patch file
    ```
    $ git diff master > pseudomanual.patch
    ```

1. Checkout your master branch and observe that the `cat.py` file is back to the original.
    ```
    $ git checkout master
    $ cat cat.py
    ```

1. Create a new empty repo on github.
    Then connect the `lab-cat` folder to this new repo by running
    ```
    $ git remote rm origin # this was already set by the submodule
    $ git remote add origin <your_url>
    $ git push origin master
    ```
    Observe on github that only your `master` branch exists remotely, and your `pseudomanual` branch does not.

**PARTNER STEPS:**

1. Copy your partner's patch file to your current folder.
    By default, every user on the lambda server has read access to every other user's home folder.
    So you should be able to run a command something like
    ```
    $ cp /home/partner_user_name/lab-coding-agents/lab-cat/pseudomanual.patch ./partner.patch
    ```
    Make sure that you don't clobber your own patch file in the command above,
    or you'll have to regenerate it.

1. Apply your partner's patch file to a new `partner` branch with the commands
    ```
    $ git checkout -b partner
    $ git apply partner.patch
    ```
    > **NOTE:**
    > It is important that you are currently on the master branch when you create `partner`, ot the `git apply` will fail.

1. Observe that the contents of your repo's `partner` branch have changed to match your partner's `pseudomanual` branch, but that the file is not yet committed.
    ```
    $ cat cat.py
    $ git status
    ```

1. Commit and push the changes.
    Because these changes were made by your partner, you should specify their email and username with `--author`:
    ```
    $ git commit --author="Parner Name <email@gmail.com>" -m 'update with patchfile'
    $ git push origin partner
    ```
    Then observe in the github interface that it shows that your partner made the commit and not you.

**WTF?!**

1. Observe for a second that you did not need to enter your partner's github password in order to register a commit by them.

    Actually, you can register a commit from any user.
    The commands below will add commits from Linus Torvalds to your `lab-cat` repo:
    ```
    $ echo '<!-- linux sux, microsoft rules -->' >> README.md
    $ git add README.md
    $ git commit --author="Linus Torvalds <torvalds@linux-foundation.org>" -m 'linus'
    ```
    and the following will add a commit from Guido van Rossum:
    ```
    $ echo '<!-- rust is the best! -->' >> README.md
    $ git add README.md
    $ git commit --author="Guido van Rossum <guido@python.org>" -m 'guido'
    ```
    View your repo on github, and you will see both Linus and Guido as contributors.

    (Sorry for lying to you all earlier---Linus and Guido did not actually help write this lab.)

    Why is github so insecure?!
    Because "git is not github".
    Git is a *distributed* version control system and github is just one of the possible interfaces.
    Git repos are allowed to exist anywhere in the world, not just on github, and so github cannot control people's identities in a centralized fashion.

    On major projects like the Linux Kernel, it is very important for everyone to be identified properly.
    Git supports decentralized identify verification through *public key cryptography* and *signing* of git commits.
    (The [git-scm.com](https://git-scm.com/book/ms/v2/Git-Tools-Signing-Your-Work) contains the official documentatation.)
    But github does not enforce this type of identity verification.

1. It turns out that github also cannot enforce anything about the dates that commits were created.
    If you run this simple bash script, you'll add 1000 commits to this repo from random times over the past year.
    This will make your git contribution chart on your account homepage look super impressive:
    ```
    for i in $(seq 1 1000); do
      echo $i >> log.txt
      git add log.txt
      export GIT_AUTHOR_DATE="$(date -d "-$((RANDOM % 365)) days" --rfc-email)"
      export GIT_COMMITTER_DATE="$GIT_AUTHOR_DATE"
      git commit -qm "commit $i"
    done
    ```

## Part 3: The coding agent

Okay, so far we have seen:
1. LLMs can write code
2. patchs can update code in repos

The way coding agents work is to just ask the LLM to directly create the patch for us!

<!--
This lab is in under development.
Everything below is notes for the future.

**Exercise:** make the patch brittle. Edit a line the patch touches and re-try:

```
$ git checkout cat.py
$ sed -i 's/import sys/import os/' cat.py
$ git apply fix.patch
error: patch failed: cat.py:1
error: cat.py: patch does not apply
```

This is the exact failure mode real coding agents hit. `committe` solves it with fuzzy-matching retries.

Commit it:

```
$ git add cat.py
$ git commit -m "fix O(n) memory in cat.py"
```

---

## Part 3: automate with `committe`

Look at `committe.sh` in the repo. It wraps exactly the workflow above:

- `committe-mkpatch` → `$llm_command -s "$(committe-prompt)"` → saves to `$(committe-patchfile)` = `.git/committe-patchfile`
- `committe-apply` → `git apply`, falls back to `git-apply-fuzzy`
- `committe-commit` → commits with a `[geni]` tag

Source it (must be `source`, not `./`, so the functions land in your shell):

```
$ source committe.sh
```

Set the llm command if not already set (skip if your `.bashrc` defines `llm_command`):

```
$ llm_command='claude -p'
```

Run the whole agent end-to-end:

```
$ git checkout cat.py                     # reset the bug
$ committe "fix cat.py to use O(1) memory"
```

Watch it:
1. send the request + `committe-prompt` system prompt to the LLM
2. write the returned patch to `.git/committe-patchfile`
3. `git apply --index` it
4. commit with the tag

Verify:

```
$ git log -1 --oneline
1a2b3c4 [geni] fix cat.py to use O(1) memory
$ ./test.sh
PASS
```

Peek under the hood:

```
$ cat .git/committe-patchfile     # the raw LLM output
$ git show HEAD                   # the commit it made
$ committe-prompt                 # the system prompt it used
```

**Note:** `committe-apply` uses `git apply --index`, so the patch is staged automatically; no separate `git add` needed.

---

## Part 4 (optional): prompt engineering

The `committe-prompt` function defines the system prompt. Have students:

```
$ committe-prompt > myprompt
$ vim myprompt      # modify instructions (e.g. "always use numpy")
```

then re-source and rerun `committe` to see output change. Point out: **a coding agent is 90% prompt engineering + 10% `git apply`**.

---

## Part 5: real-world practice

Clone last week's homework repo (or the `continuous-integration` repo) and:

```
$ cd ~/myrepo
$ committe "fix all flake8 errors"
$ committe "add type annotations to every function"
$ ./test.sh
```

Observe failure modes:
- LLM returns prose instead of a patch → `git apply` fails
- LLM hallucinates context lines → `git-apply-fuzzy` retries
- Patch touches a file that doesn't exist → hard failure

---

## Submission

1. Screenshot of `committe` successfully applying a self-generated patch to a repo of your choice.
2. Output of `git log --oneline` showing the `[geni]` commits.
3. One paragraph: in your own words, why does `committe` re-run the LLM if `git apply` fails the first time?

```
$ git log --oneline
... [geni] fix ...
-->
