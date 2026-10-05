# Review aid PR skill

```shell
npx skills add koterpillar/review-aid
```

## What the skill does

A large refactoring pull request (PR, a set of proposed code changes) mixes
two kinds of changes. One kind is a rename or a move. The other kind is a
real, meaningful change. When a reviewer reads the PR, the renames fill most
of the diff (the list of changed lines). The reviewer must search through the
renames to find the real changes. This is slow and it increases the chance
that the reviewer misses a real change.

The `review-aid` skill creates a second PR that removes the rename noise.

<table>
<tr><td>main</td><td>←</td><td colspan="5" align="center">(large diff)</td><td>←</td><td>feature</td></tr>
<tr><td>main</td><td>←</td><td>(mechanical changes)</td><td>←</td><td>review aid</td><td>←</td><td>(small diff)</td><td>←</td><td>feature</td></tr>
</table>

In the original PR, feature branch carries the large diff directly against main branch.
The large diff has the renames and the real changes together.

The skill adds a review aid branch. It adds the mechanical changes like renames
on top of the main branch. The diff from it to the feature branch is now only
real changes without the rename noise.

It raises a temporary PR targeting the review aid branch, where a reviewer can
read the small diff without the noise. The original PR stays unchanged. The new
PR is marked as "do not merge" and can be removed with the temporary branches
after the feature is merged.

## Examples

<table>
<tr><th>Before</th><th>After</th></tr>
<tr><td>

```diff
-public class RequestData {
+public class Request {
   private Map<String, String> headers;
   private String body;
+  private Map<String, String> trailers;
 }
```

</td><td>

```diff
 public class Request {
   private Map<String, String> headers;
   private String body;
+  private Map<String, String> trailers;
 }
```

</td></tr>
<tr><td>

```diff
-import jest, { it } from "jest";
+import vi, { it } from "vitest";

 describe("parser", () => {
-  jest.mock("./tokenizer");
-  it("parses a valid token", () => {
+  vi.mock("./tokenizer");
+  it("parses a valid token", async () => {
     expect(parse("OK")).toBe(true);
   });
 });
```

</td><td>

```diff
 describe("parser", () => {
   vi.mock("./tokenizer");
-  it("parses a valid token", () => {
+  it("parses a valid token", async () => {
     expect(parse("OK")).toBe(true);
   });
 });
```

</td></tr>
</table>
