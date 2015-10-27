# gtfc
git for cluster sync, naive synchronization over ssh.

# How it works

Use a kqueue watcher service to trigger gtfc-addandpush if files have changed.

Instead of lsyncd (which deletes the file without keeping history)

# Properties

Property                                   | Description
-------------------------------------------|------------------
Centralized aspect                         | Only the first node with unique/latest data will be central until it completes a push.
Decentralized/distributed storage          | Kind of. When all nodes are up to date, there is no central store because git repositories are mirrors. Akin to RAID1 mirror, but not to RAID5 span.
Decentralized/distributed transfers        | Kind of. Nodes can push to any other nodes.
Chunked data transfers from multiple hosts | No.
Security?                                  | Pretty good.
Optimal use                                | Good for small files or text.
Limitations                                | Bad for large files or binary. Sync based on git commits, not files. Since we use authorized_keys2 to let peers in, make sure to have different users for different repos if you need to restrict peers.

# Usage

## 1. SSH into alpha and bootstrap the host

```
local$  ssh alpha
alpha$  gtfc-bootstrap ~/myrepo
```

## 2. SSH into beta with forwarding of the authentication agent connection

```
local$  ssh -A beta
beta$  gtfc-join alpha ~/myrepo
```

# Can some hosts be read-only?

Yes.

Set the `command` in authorized_keys2, replacing `gtfc-shell` with `gtfc-shell.denywrite`
```
