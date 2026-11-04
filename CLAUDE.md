# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Status (as of 2026-11-04)

Tom chose to keep this guide, and PR #1 made small fixes. The full refresh in `PLAN.md` hasn't started. It waits until Tom's lab work is done (the AWS lab and `cka-d-cluster-builder-multipass`), and it only starts when Tom asks. The guide is still unfinished: see "What's unfinished" below.

## What this repo is

Tom Dean's tutorial on using `kubectl` imperatively to generate YAML manifests, written in 2023:

- `README.md` is about 3,000 lines, much of it sample output. It covers `kubectl` help and `api-resources`, imperative Pods, and `--dry-run=client --output=yaml` manifests for Namespaces, Pods, Deployments, and Services, then scaling, labels, logs, exporting objects, deleting them, and 10 practice exercises.
- `manifests/` has example manifests from the tutorial.
- `solutions/` has one file per exercise, plus `solution-template.md`.

There's no license yet, and Tom hasn't chosen one. See `PLAN.md`.

Related repos under `southsidedean`:

- `cka-d-cluster-builder-lab` built the cluster this guide's examples ran on. Its docs are the source for current Kubernetes versions.
- `cka-d-cluster-builder-multipass` and `kubeadm-cluster-builder-aws-terraform` are the other labs in that family.
- `southsidedean` is the profile repo. Its README lists this guide under "Guides and labs".

`deploy-kind-using-podman-ubuntu` used to be this guide's KIND prerequisite. It was archived on 2026-11-04, and the links now point to KIND's quick start. Don't link to it or to the other archived 2023 guides (the Confluence guides and `deploy-kind-using-podman-macos`).

## What's unfinished

Checked on 2026-11-04. The README was read closely up to about line 1,470, plus the exercises and the opening lines of the later sections, so read the rest before relying on it.

- Solutions: `solution-exercise-1.md` looks written but hasn't been checked against a cluster. `solution-exercise-9.md` has headings and empty code blocks. The others (2 to 8, and 10) have no solution steps. Nothing in the README warns readers about this.
- Section intros: logs, exporting objects, and deleting objects have no intro text under their headings. Check the other later sections too.
- Sample output: all from 2023, on a two-node Kubernetes 1.26.1 `kubeadm` cluster on Ubuntu 22.04. A note in the prerequisites says so.

## Testing

Every sample in the README comes from a real run. Never invent output or edit it to look newer.

Nothing has been re-run on a cluster. On 2026-11-04, with `kubectl` 1.37.2 and no cluster, four of the five `--dry-run=client` commands that start a line ran. `kubectl run` failed because it needs a server to reach, even with `--dry-run=client`. Generated YAML no longer includes `creationTimestamp: null`, so the samples and `manifests/` will change. A grep for the removed flags `--export` and `--generator`, and for old `v1beta` API versions, found nothing.

- Ask Tom before building a cluster or changing one. Check `kubectl config current-context` first, because the current context may belong to other work. For client-only commands, set `KUBECONFIG=/dev/null`.
- The test machines are shared with other work, so leave alone any cluster or VM you didn't create.
- The guide says either a `cka-d-cluster-builder-lab` cluster or KIND will work, but that's untested on KIND, and its `curl <POD_IP>` steps may not reach pods from outside a KIND cluster. Test it or change the claim.

## Conventions

- Never invent facts. The public email is tom@southsidedean.com. Review prose with the `humanizer` skill, and keep Tom's voice and the guide's structure. Ask Tom before describing their exam or certification status.
- Work goes through a branch and a PR, and Tom merges. Ask before committing, pushing, or opening PRs, and check `git status` and the current branch first. This repo has no CI or branch rules, so nothing checks a PR automatically.
- Private details, such as test host names, go in `CLAUDE.local.md`, never in this file or `PLAN.md`. `.git/info/exclude` keeps `CLAUDE.local.md`, `.claude/`, `.mcp.json`, `.backups/`, and `.DS_Store` out of git.
- Use they/them for Tom.
