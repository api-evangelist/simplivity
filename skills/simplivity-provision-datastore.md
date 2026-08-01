---
name: Provision and manage a SimpliVity datastore
description: Create a datastore, bind a backup policy, resize it, and share it to standard ESXi hosts using the HPE OmniStack REST API.
api: openapi/simplivity-omnistack-openapi-original.json
operations: [createDatastore, listDatastores, setDatastorePolicy, resizeDatastore, shareDatastore]
---

# Provision and manage a SimpliVity datastore

Uses the HPE OmniStack REST API with an OAuth2 bearer token (`write` scope for
mutations). Creates and changes run asynchronously — poll `getTask`.

## Steps

1. **Create the datastore** — call `createDatastore` (`POST /datastores`) with a
   name, target omnistack cluster, size, and the backup policy to attach.
2. **Confirm it exists** — call `listDatastores` (`GET /datastores`) using
   `offset`/`limit` paging and `fields` to project the attributes you need.
3. **(Re)bind a policy** — call `setDatastorePolicy`
   (`POST /datastores/{datastoreId}/set_policy`) to change the governing backup
   policy.
4. **Resize** — call `resizeDatastore`
   (`POST /datastores/{datastoreId}/resize`) to grow the datastore; poll
   `getTask` for completion.
5. **Share to hosts** — call `shareDatastore`
   (`POST /datastores/{datastoreId}/share`) to expose it to a standard ESXi host
   (use `getStandardHosts` to enumerate eligible hosts; `unShareDatastore` to
   revoke).

## Notes
- Deleting a datastore (`deleteDatastore`) requires it be empty; a 409-class
  conflict indicates in-use resources.
- Watch the returned task's `error_code`/`message` on `FAILED` states.
