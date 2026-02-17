# UniqLearn — Organization Configuration

This repository contains organization-level workflows and community health files for the UniqLearn GitHub organization.

## Org-Level Workflows

### Planning Gate Dispatcher (`.github/workflows/planning-gate-dispatch.yml`)

Listens for project board status changes and triggers the Planning Gate on the Platform repo.

**Trigger:** `projects_v2_item` edited event (fires when any project field changes on any issue in the org)

**Conditions (all must be true):**
- The changed field is **Status**
- New status value is **"Planning"**
- Issue type is **"Feature"**
- Issue belongs to the **Platform** repo

**Action:** Dispatches `workflow_dispatch` to `UniqLearn/Platform` with the issue number, which runs `planning-gate.js` to generate a structured plan comment on the issue.

**Required secret:**

| Secret | Purpose |
|--------|---------|
| `PLANNING_GATE_TOKEN` | Fine-grained PAT with Actions (write), Issues (read), Projects (read) scoped to Platform and .github repos |

**Flow:**

```
Developer changes Status → "Planning" on issue sidebar
    ↓
GitHub fires projects_v2_item event (org-level)
    ↓
This workflow catches it
    ├── Queries project item via GraphQL
    ├── Checks: Type = Feature? Status = Planning? Repo = Platform?
    └── Dispatches workflow_dispatch to UniqLearn/Platform
         ↓
Platform planning-gate.yml runs planning-gate.js
         ↓
4-section plan comment appears on the issue
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
     - Issues: Read
     - Projects: Read (organization)
3. Add the token as a secret named `PLANNING_GATE_TOKEN` at:
   https://github.com/UniqLearn/.github/settings/secrets/actions
