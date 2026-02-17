# UniqLearn — Organization Configuration

This repository contains organization-level workflows and community health files for the UniqLearn GitHub organization.

## Org-Level Workflows

### Planning Gate Dispatcher (`.github/workflows/planning-gate-dispatch.yml`)

Polls the org project board and triggers the Planning Gate on the Platform repo when a Feature issue is moved to "Planning" status.

**Trigger:** Scheduled every 5 minutes (also supports manual `workflow_dispatch`)

> **Why polling?** The `projects_v2_item` webhook event is NOT a supported GitHub Actions trigger — it only works as an organization webhook, not in `on:` blocks. Schedule-based polling is the reliable alternative.

**Conditions for dispatch (all must be true):**
- Item is on org project #11 ("Vibe Coding OS")
- Status field is **"Planning"**
- Issue type is **"Feature"**
- Issue belongs to the **Platform** repo
- Issue does NOT have the `planning-gate-dispatched` label (dedup)

**Dedup mechanism:** After dispatching, the workflow adds a `planning-gate-dispatched` label to the issue to prevent re-dispatching on subsequent polls. To re-trigger the planning gate for an issue, remove this label.

**Action:** Dispatches `workflow_dispatch` to `UniqLearn/Platform` with the issue number, which runs `planning-gate.js` to generate a structured plan comment on the issue.

**Required secret:**

| Secret | Purpose | Scope |
|--------|---------|-------|
| `PLANNING_GATE_TOKEN` | Fine-grained PAT for project queries, cross-repo dispatch, and issue labels | Actions (write), Issues (write), Projects (read) on Platform and .github |

**Flow:**

```
Schedule: every 5 minutes
    |
    v
Poll project #11 via GraphQL
    |
    v
Filter: Status=Planning, Type=Feature, Repo=Platform, No dedup label
    |
    v
For each matching issue:
    ├── Dispatch workflow_dispatch to UniqLearn/Platform planning-gate.yml
    └── Add 'planning-gate-dispatched' label to prevent re-dispatch
         |
         v
    Platform planning-gate.yml runs planning-gate.js
         |
         v
    4-section plan comment appears on the issue
```

**Manual dispatch:**

```bash
# Dispatch for a specific issue (bypasses project board scan)
gh workflow run planning-gate-dispatch.yml \
  --repo UniqLearn/.github \
  --field issue_number=54

# Or dispatch directly on Platform
gh workflow run planning-gate.yml \
  --repo UniqLearn/Platform \
  --field issue_number=54
```

## Setup

### Creating the `PLANNING_GATE_TOKEN`

1. Go to https://github.com/settings/tokens?type=beta
2. Create a fine-grained token:
   - **Token name:** `planning-gate-dispatcher`
   - **Resource owner:** UniqLearn
   - **Repository access:** Select `Platform` and `.github`
   - **Permissions:**
     - Actions: Read and write
     - Issues: Read and write (needed for labels)
     - Projects: Read (organization)
3. Add the token as a secret named `PLANNING_GATE_TOKEN` at:
   https://github.com/UniqLearn/.github/settings/secrets/actions

### Re-triggering the Planning Gate

If you need to re-run the planning gate for an issue that was already processed:

1. Remove the `planning-gate-dispatched` label from the issue
2. Wait for the next 5-minute poll cycle (or trigger manually)
