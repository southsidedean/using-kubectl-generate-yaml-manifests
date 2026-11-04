# using-kubectl-generate-yaml-manifests plan

Tom Dean, updated 2026-11-04

The guide stays, but it needs finishing. The full refresh waits until the lab work is done. It's the largest job left among the old guides, so it's split into phases that can each be a separate PR.

## Done

PR #1 (2026-11-04) pointed the KIND links to KIND's quick start, added a note that the sample output dates from 2023, removed three `***Transition***` markers, and wrote the summary.

## Could do now, without a cluster

1. Warn readers about the blank solutions with one line above the exercises, until the solutions are written. Tom hasn't decided on this yet.
2. Fix the typos and wording:
   - "declarively", "swiches" (twice), and "the the".
   - `nginx-latest` should be `nginx:latest`, in both the tutorial and Exercise 4.
   - "Create the `my-nginx-deployment` namespace manifest" should say `my-nginx-namespace`.
3. Check the references. The kubectl cheat sheet link now redirects to the quick reference page. The Medium article returned 403 to an automated check, so check it in a browser.
4. Review the summary written on 2026-11-04 with the `humanizer` skill.

## Full refresh

Start after the lab work, in this order:

1. Get a cluster from the current `cka-d-cluster-builder-lab` build, or from the Multipass builder once it has a release. Tom approves building it.
2. Read the whole README, since only the first half has been read closely.
3. Re-run every command, replace the sample output, and update `manifests/` to match. Expect `creationTimestamp: null` to be gone from generated YAML, and the help, `api-resources`, and `kubectl explain` output to have changed.
4. Write the missing section intros (logs, exporting, deleting, and any others found) and update the prerequisites note.
5. Check solution 1 on a cluster, finish 9, and write 2 to 8 and 10. Remove any warning added above.
6. Settle the open questions below, add the license, and do a final `humanizer` pass.

## Decisions

| Question | Decision |
| --- | --- |
| Keep or archive | Keep (Tom, 2026-11-04) |
| Timing | Small fixes done; the full refresh waits for the lab work (Tom, 2026-11-04) |
| Sample output | Real runs only |

## Open questions

- License: the repo has manifests as well as prose. The family's rule is MIT for code and CC BY 4.0 for prose-only guides, so this could be MIT, CC BY 4.0, or both.
- KIND: the guide says KIND works, but that's untested, and its `curl <POD_IP>` steps may not work from outside a KIND cluster. Test it, adjust the steps, or drop KIND as an option.
- Exercises: Exercise 4 asks for 5 replicas, but Exercise 6 says to scale from 3. Exercise 8 asks for a Secret, which the tutorial doesn't appear to teach. Exercises 3 to 6 reuse the tutorial's object names, so they may collide unless the tutorial's cleanup was done first.
- Introduction: it says "I'm currently studying" for the exams, which was true in 2023. Tom decides how it should read now.
- Repeated paragraph: the note about Adrien Trouillaud's article appears twice, the second time under "AGAIN". Check whether Tom wants to keep both.
- Kubernetes version: the labs track the latest release, so choose between naming a version in the text or only in the sample output.
