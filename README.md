# gtfc
git for cluster sync over ssh: on kqueue, add, commit, push, merge.

## How it works

Use a kqueue watcher service to trigger `gtfc-push` if files have changed.
There's an example in `gtfc-watch` which launches `gtfcwatch.py`

## Properties

Property                                   | Description
-------------------------------------------|------------------
Centralized aspect                         | Only the first node with unique/latest data will be central until it completes a push.
Decentralized storage                      | Yes. When all nodes are up to date, there is no central store because git repositories are mirrors. Akin to RAID1 mirror, but not to RAID5 span.
Distributed storage                        | No, files are not split/striped.
Decentralized/distributed transfers        | Yes. Nodes can push to any other nodes.
Chunked data transfers from multiple hosts | No.
Security?                                  | Pretty good, but only as good as your SSH config.
Optimal use                                | Good for small files, best with text.
Limitations                                | Bad for large files or binary. Sync based on git commits, not files. Since we use authorized_keys2 to let peers in, make sure to have different users for different repos if you need to restrict peers.

## Usage

### 1. SSH into alpha and bootstrap the host

```
local$  ssh alpha
alpha$  gtfc-bootstrap ~/myrepo
```

### 2. SSH into beta with forwarding of the authentication agent connection and clone from alpha

```
local$  ssh -A beta
beta$  gtfc-join alpha ~/myrepo
```

### 3. On either, edit a file and push manually

```
alpha$  echo `date` `hostname` > ~/myrepo/test
alpha$  gtfc-push
```

### 4. If a node is rebooted

You probably want to pull changes at boot or in a boot script.

```
beta$  gtfc-pull
```

### 5. To watch for automatic changes, use kqueue watchdog or inotify, etc

```
alpha$  gtfc-watch
beta$  gtfc-watch
```

## Can some hosts be read-only?

Yes. Since the we force a command on ssh connections, we can just use a gtfc-shell that doesn't allow receiving a push.

Set the `command` in authorized_keys2, replacing `gtfc-shell` with `gtfc-shell.denywrite`

# Similar but different tech

lsyncd + rsync (which deletes the file without keeping history)
