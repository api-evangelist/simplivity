---
name: Create a backup policy and apply it to a VM
description: Create a SimpliVity backup policy, add a scheduling rule, review the proposed schedule, and assign the policy to a virtual machine using the HPE OmniStack REST API.
api: openapi/simplivity-omnistack-openapi-original.json
operations: [createPolicy, createRule, getPolicyScheduleReport, listPolicies, setPolicyForVm]
---

# Create a backup policy and apply it to a VM

Uses the HPE OmniStack REST API with an OAuth2 bearer token (`write` scope
required for creates/assignments). Long-running changes return a task — poll
`getTask` for the terminal state.

## Steps

1. **Create the policy** — call `createPolicy` (`POST /policies`) with a policy
   name. This defines the container for one or more backup rules.
2. **Add a rule** — call `createRule` (`POST /policies/{policyId}/rules`) to set
   frequency, retention, destination (local cluster or external store), and
   application-consistency options.
3. **Review the schedule** — call `getPolicyScheduleReport`
   (`GET /policies/{policyId}/schedule_report`) to confirm the resulting backup
   cadence before rollout. You can also dry-run edits with
   `reportOnProposedRulesCreation`/`reportOnProposedRulesEdits`.
4. **Find the policy** — call `listPolicies` (`GET /policies`) to resolve the
   `policyId` if you did not retain it.
5. **Assign to a VM** — call `setPolicyForVm`
   (`POST /virtual_machines/{vmId}/set_policy`) to bind the VM to the policy;
   poll `getTask` until `COMPLETED`.

## Notes
- `getPolicyVirtualMachines` and `getPolicyDatastores` are deprecated — prefer the
  VM- and datastore-side lookups.
- 403 indicates a missing `write` scope or role assignment; 404 an unknown policy
  or VM id.
