# Lab: Coding Agents

In this lab you will create a simple coding agent based on `llm` or `dic`.
The agent will still be missing a few features from tools like Claude Code,
but is a simple and reliable "workhorse" that you can feel free to use on any assignment in this class.

<img src=img/xkcd.png width=300px />

> **NOTE:**
> Everything in this lab works with `llm` and the free tier qwen models provided by groq.
> But I've also created a public API key that everyone in this class can share so that you can all use fancier newer models without paying.
> You can get access to the API key by running the command
>
> ```
> $ export OPENROUTER_API_KEY='sk'-'or'-'v1-7568a8484b8'55a500d2ac7920fa31efe59f1d53f3b060d914290d543ab36e337
> ```
>
> > **DOUBLE NOTE:**
> > Observe the weird single quotes `'` to the right of the equals above.
> > Github performs *secret scanning* checks on all files committed to github looking for API keys that match a regex.
> > Normally, people do not want to actually upload API keys to github because then everyone in the world has access to them.
> > But in this case I actually do want to upload the API key.
> > To avoid the regex check that github uses, I added these `'`.
> > *Semantically* this has the same meaning as the shell without the `'`,
> > It is only a *syntactic* difference.
>
> Then with `OPENROUTER_API_KEY` in your environment,
> you should be able to use any openrouter model from either `dic` or `llm`.
> From `llm`, you'll have to pip install the `llm-openrouter` extension then you can run the `llm models` command to view all of the models available (which is *every* model that exists).
> `dic` has openrouter models enabled without any extensions,
> and you can run with the latest deepseek model by passing the `-m openrouter+deepseek` flag:
> ```
> $ dic -m openrouter+deepseek 'hello'
> ```
> The API key has $5 associated with it, which is between 5k-50k API calls depending on how large the context window is.
> So how long this key lasts for depends on how much you all are using it.

Your agent will rely on git *patch files*.
Patch files are core to the Linux and Python development process,
and you'll notice that Linus and Guido both helped write portions of this lab.

<img src=img/contrib.png width=200px />

Linus has historically had a very conservative approach to adopting new technologies in the linux kernel and is famous for his brash personality and calling people out for writing low-quality code.
For example:

<img src=img/linus1.png width=400px />

And when Linus started accepting AI code in the Linux kernel,
many saw this as the official turning point that AI coding agents are here to stay.
If Linus thinks AI is good enough for the Linux kernel,
it's probably good enough for whatever types of projects you are working on.

<img src=img/linus2.png width=300px />

You'll need a partner for Part 2 of this lab to practice the [Linux patchfile contribution process](https://docs.kernel.org/process/applying-patches.html).
This process forms the foundation for how AI agents write code,
but is slightly more technical than the github pull request.
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

## Part 1: pseudomanually fixing

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
    > It is important that you are currently on the master branch when you create `partner`, or the `git apply` will fail.

1. Observe that the contents of your repo's `partner` branch have changed to match your partner's `pseudomanual` branch, but that the file is not yet committed.
    ```
    $ cat cat.py
    $ git status
    ```

1. Commit and push the changes.
    Because these changes were made by your partner, you should specify their email and username with `--author`:
    ```
    $ git commit --author="Partner Name <email@gmail.com>" -m 'update with patchfile'
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
2. patches can update code in repos
A coding agent is just those two facts glued together: **tell the LLM to write the patch, then `git apply` it**.

In this part you will write the agent.
We'll call it `committe` (Latin for "put together" and the origin of English "commit") in the spirit of `dic`---we are casting spells to force the llm to do our bidding.

### Generating the Patch

The first key idea is that the prompt must specify the output format.
Below is the prompt I use wrapped in a bash function.
Notice that the prompt:
1. starts high level,
2. then provides an examples,
3. then provides detailed rules.
This 3-stage prompt format is a good template for designing your own prompts.

```bash
function committe-prompt() {
    cat <<EOF
You are a coding agent.
The user describes a change they want made to a git repository.
You respond with:
1. a commit message (Tim Pope style)
2. a patch.
Here is an example:

\`\`\`
fix the foobar bug

diff --git a/path/to/file b/path/to/file
--- a/path/to/file
+++ b/path/to/file
@@ -<old_start>,<old_count> +<new_start>,<new_count> @@
 context line
-removed line
+added line
 context line
\`\`\`

Rules:
- No other content.
    - Do NOT wrap your response in markdown code fences.
    - Do NOT include any prose other than the commit message
- The commit message uses Tim pope style
    - imperative header (50 char max)
    - optional body explaining the changes
        - should be used only on complex patches
- Use standard unified diff syntax with '--- a/...' and '+++ b/...' headers.
    - For new files use '--- /dev/null' and '+++ b/path'.
    - You must also specify the mode of the new file
      (Add the text "new file mode 100644")
    - For deleted files use '--- a/path' and '+++ /dev/null'.
- The patch will be applied with \`git apply --recount\`
    - Hunk line numbers do not have to be exact,
      but the context lines must be recognizable in the current file.
    - Include 2-3 lines of unchanged context around each change.
    - These context lines must exactly match the original document.
      (Including whitespace, quotation marks, and other punctuation.)
- Prefer small, focused patches.
- State a structural change the way git states it, never as content.
    - To move a file, state the rename and no hunks.
      For example:
          diff --git a/old b/new
          similarity index 100%
          rename from old
          rename to new
      A move that also edits the file states those two rename lines and
      then the hunks, which are the change against the old contents.
    - To delete a file, state the mode and no hunks:
          diff --git a/old b/old
          deleted file mode 100644
    - To change a file's mode and nothing else:
          diff --git a/script b/script
          old mode 100644
          new mode 100755
    - A new file states its mode: 100644, or 100755 when it is executable.
    - A symlink is a new file of mode 120000 whose one added line is the
      path it points at.
- If the change cannot or should not be made yet -- the request is
  ambiguous, the tree does not support it, or you need a decision the user
  has not made -- write no patch and reply with your question alone.
    - It is printed and nothing is committed.

Use the following information to help you write the code:

$ git ls-files
$(git ls-files)
EOF
}
```

Create a file `committe.sh` and place the `committe-prompt` function inside of it.
In order to have access to this function in the shell, you'll need to source your script:
```
$ source committe.sh
$ committe-prompt
```
(Observe above that functions are called just like regular programs.)

Now we can write another function that invokes `dic` (or `llm`) and creates the patchfile:

```bash
function committe-mkpatch() {
    dic -s "$(committe-prompt)" "$@" > "./.git/committe-patchfile"
}
```
Add this function to your `committe.sh` script and re-source it.

Inside of the `lab-cat` repo's master branch (where the `cat.py` file has not been fixed), you can now generate a patch by running a command like
```
$ committe-mkpatch <<EOF
$(files-to-prompt . .github)

Fix the python.
EOF
```
Then inspect the patch with
```
$ cat .git/committe-patchfile
```

There are a few subtlties to observe:

1. The heredoc applied to `committe-mkpatch` gets passed over to `dic` inside of it,
    and so `dic` will write a patchfile to stdout, and output redirection will place this patchfile at `./.git/committe-patchfile`.
    A file inside the `.git` folder was chosen because git ignores these files, and so the patch will not accidentally end up being committed to the repo.
    It is standard for tools that work with git to place their temporary files in the `.git` repo like this.

1. The `"$@"` inside `committe-mkpatch` passes all the command line arguments to the `dic` program.
    These means we can use `committe-mkpatch` just like we would `dic`.
    A common pattern is to first start a conversation with `dic` asking questions,
    then switch over to `committe-mkpatch` when we are ready to code:
    ```
    $ dic <<EOF
    $(files-to-prompt . .github)
    what is the purpose of this repo?
    EOF
    $ committe-mkpatch -c 'implement the fix'
    ```

### Applying the patch

We're almost done.
The last step is to apply the patch automatically.
The function below shows how to do that:

```bash
function committe-apply() {
    # First we apply the patch.
    # Notice that we have added the --recount and --ignore-whitespace flags.
    # These allow git apply to be more flexible when applying the patch,
    # and so small typos (which llms are likely to do) will not cause the patch to fail.
    # It is still possible, however, for the patch to fail if the llm made major mistakes, which happens on occasion.
    if ! git apply --index --recount --ignore-whitespace '.git/committe-patchfile'; then
        echo 'git apply failed'
        return 1
    else

    # The git apply command ignores the commit message at the top of the patchfile.
    # Now we extract that message with sed.
    local msg
    msg="$(sed -e '/^diff --git/,$d' "./.git/committe-patchfile")"

    # We commit specifying the --author flag and tagging the message.
    # Both of these modifications make it easy to idenitfy which commits were made automatically.
    git commit -m "[committe] $msg" --author="committe <committe@committe.ai>"
}
```

Now we are ready to define the whole agent, which is just:

```bash
function committe() {
    committe-mkpatch "$@"
    committe-apply
}
```

Add the `committe-apply` and `committe` functions to your `committe.sh` script and re-source it.

### Running it

There are several ways you can use `committe` to implement `lab-cat`:

1. This problem is simple enough that any model is able to 1-shot solve it.
    (1-shot refers to the fact that the model can solve the problem in one run without breaking it into parts or back-and-forth converation.)
    ```
    $ git checkout master
    $ git checkout -b 1shot
    $ committe <<EOF
    $(files-to-prompt . .github)
    fix the python
    EOF
    $ cat cat.py
    ```

1. More complicated problems require a back-and-forth conversation to solve.
    `committe` integrates nicely with `dic` (or `llm`).
    ```
    $ git checkout master
    $ git checkout -b conversation
    $ dic <<EOF
    $(files-to-prompt README.md)
    what is this project about?
    EOF
    $ dic -c <<EOF
    what files do I need to provide to solve the problem?
    EOF
    $ dic -c <<EOF
    $(files-to-prompt .github cat.py)
    EOF
    $ committe -c 'implement it'
    $ cat cat.py
    ```
    Recall that the `-c` flag stands for "continue" the conversation,
    so the final call to `committe` will have all the context from the previous conversation available to write the code.

## Submission

Push your `conversation`, `1shot`, `partner`, and `pseudomanual` branches to github.
Recall that for each branch:
1. you will have to first checkout the branches in your local repo,
2. then run `git push origin <branch>`.

Submit the url of your repo to canvas.
I will check that all branches are correctly uploaded.

---

I strongly encourage you to try to use the `committe` tool throughout the rest of the course.
The model `deepseek-v4.1-flash` (or any other SOTA model) can essentially 1shot all the code you need for the mapreduce assignment...
but you would still have to figure out how to orchestrate all the files and get them to run correctly...

If you ever need to undo a commit created by `committe`, the git incantation is
```
$ git reset --hard HEAD~1
```
