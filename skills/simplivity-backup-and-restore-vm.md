---
name: Back up and restore a SimpliVity virtual machine
description: Create an on-demand backup of a virtual machine, confirm it via the task poll, list the VM's backups, and restore from a chosen backup using the HPE OmniStack REST API.
api: openapi/simplivity-omnistack-openapi-original.json
operations: [backupVm, getTask, listBackups, getBackup, restoreFromBackup]
---

# Back up and restore a SimpliVity virtual machine

Uses the HPE OmniStack REST API. All calls require an OAuth2 bearer token
(`Authorization: Bearer <access_token>`) obtained from `POST /api/oauth/token`
(grant_type=password, read/write scopes). Tokens expire after 10 minutes of
inactivity and 24 hours absolute; TLS 1.2+ is required.

## Steps

1. **Create the backup** — call `backupVm` (`POST /virtual_machines/{vmId}/backup`)
   with the target `vmId` and a backup name/destination. Mutating operations are
   not idempotent, so do not blind-retry; instead follow the task.
2. **Confirm completion** — the operation returns a task; poll `getTask`
   (`GET /tasks/{taskId}`) until `state` reaches `COMPLETED` (or handle
   `FAILED`/`TIMEDOUT`, reading `error_code`/`message`).
3. **List the VM's backups** — call `listBackups` (`GET /backups`) filtered to the
   VM, using `offset`/`limit` to page and `sort`/`order` to order results.
4. **Inspect one backup** — call `getBackup` (`GET /backups/{bkpId}`) to verify
   size/state before restoring.
5. **Restore** — call `restoreFromBackup` (`POST /backups/{bkpId}/restore`) to
   create a new VM or replace the original from that backup; poll `getTask` again
   for the terminal state.

## Notes
- Errors surface as standard HTTP status (401 expired/invalid token, 403 missing
  scope/role, 404 unknown VM or backup). Per-operation failures appear in the
  TaskMO body, not as problem+json.
