---
name: review-aid
description: Creates a PR that aids in reviewing a large refactoring PR.
---

It is hard to read a big PR diff that does renames or migrates imports as well
as making semantic changes. The renames might dominate the diff, making it
harder to figure out the changes are genuinely important.

To create a review aid PR:

1. Identify the base/destination and source branches.
2. Identify the noise that can reduce the diff if removed.
3. Create the "aid base" branch and apply the renames.
4. Create the new PR targeting the aid base branch.

Here are the step details:

Step 1. Identifying the base and source branches

If the user gave a PR (URL, ID, or "this PR"/"current PR"), look it up via the
appropriate forge (GitHub, Bitbucket, etc.) tooling (`gh`, `twg`) or MCPs to get
its source branch and destination (base) branch.

If no PR was specified, use the current checked-out branch as the source
branch, and that branch's configured upstream/tracking branch as the base
branch. If there is no tracking branch, fall back to the repository's main
branch (e.g. `main` or `master`) as the base.

If the base/source branches are still ambiguous, ask the user to confirm
before proceeding.

Step 2. Identifying the noise.

The PR is expected to have both significant changes as well as rename noise.
Inspect the diff for the current PR. Pay attention to type, function, class and
other identifier renames, as well as moved files. Also consider replacements
with similar interfaces if the refactoring replaced, for example, one dependency
with another with a similar interface.

Remember which files or identifiers got renamed or replaced (semantically). If
there are none, stop and tell the user you can't identify any.

Examples:

```diff
-import { it } from "jest";
+import { it } from "vitest";
```

`jest` replaced with `vitest`, replacing `from "jest"` to `from "vitest"` would
help. (A simple replacement of `jest` with `vitest` might be too broad, or it
might be better for catching other similar renames.)

```diff
-type RequestData = {
+type Request = {
   headers: Map<string, string>;
   body: string;
+  trailers: Map<string, string>;
 }
 ```

`RequestData` got renamed to `Request`. There is an additional member added to
it, but that's a separate change that should be reviewed, not mechanical noise.
For the review aid PR, replacing `RequestData` with `Request` would help.

A framework or library migration often creates many rename pairs at once. For
example, a whole namespace of functions, types, or methods can change names
together. Before you write any replacement command, list every rename pair
for the PR at hand. Report the full list to the user. This list is your plan
for step 3, and it is also a record of the work.

Treat each pair in the list as its own independent, literal, global
find/replace. Run each one as its own command, with `sed` or `ast-grep`, and
verify its own contribution to the diff on its own. Do not combine several
pairs into one script. A single script that handles several pairs with
shared logic can hide a wrong replacement in one pair inside the results of
the other pairs, and this makes the error hard to find during verification.

Some old names do not map to a single new name. The correct new name depends
on the surrounding code, for example an argument, a method chain, or a branch
of an if statement. This kind of replacement is not noise removal. It is a
judgment call, and it can pick the wrong replacement for some occurrences.
Mark this kind of replacement as a separate, higher-risk category, and tell
the user about it. Verify a sample of each resulting variant by hand. Do not
rely only on the overall size of the diff to verify this kind of replacement.

Step 3. Applying renames to the base branch.

Create a new "aid base" branch from the original PR base branch. Use a separate
worktree if practical.

Then perform the replacements identified in step 2. HEAVILY prefer automatic
tools with no AI input like `sed` and `ast-grep`.

After each replacement, verify the diff between the aid base branch and the
original source branch is reduced by the noise that was identified in step 2. If
there are unexpected changes as a result of a too broad replacement, reattempt
the replacement with a more precise scope (file limit, more precise search
string even if there will be several of them needed instead of one).

Only if the precise replacement is not possible with automatic tools, you may
perform the replacements manually. Still verify that the replacement is correct.

After verification, commit the changes. The commit hooks will likely fail
because the mechanical changes can break linting, compilation or typechecking -
ignore the hooks and commit with `--no-verify`. As a commit message, put exact
replacements done - commands and scope.

Step 4. Create the review aid PR.

Back in the original PR worktree (likely the main one for the repository),
create a new "aid source" branch and point it to the same commit as the original
PR source branch.

Check the diff between the aid base branch and the aid source branch. It should
be shorter and free of noise identified in step 1. If not, go back to step 2 or
3.

Create a new PR from aid source to aid base branch. Use the repository
conventions, but indicate "DO NOT MERGE" and set the PR as draft. In the
description, say that this is a review aid for the original PR (link it), which
replacements were made in the aid base branch and why.
