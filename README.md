# gtfc
git for cluster sync, naive synchronization (pull only) over ssh every minute (with crontab)

# How it works

Using cron to schedule a task that: in a git repo, for each git remote, perform a git pull.

# Properties

Property                                   | Description
-------------------------------------------|------------------
Centralized aspect                         | Only the first node with unique/latest data will be central until another completes a pull.
Decentralized/distributed storage          | Kind of. When all nodes are up to date, there is no central store because git repositories are mirrors. Akin to RAID1 mirror, but not to RAID5 span.
Decentralized/distributed transfers        | Kind of. Nodes can pull from any other nodes that are ahead.
Chunked data transfers from multiple hosts | No.
Optimal use                                | Good for small files or text.
Limitations                                | Bad for large files or binary. Sync based on git commits, not files. Since we use authorized_keys2 to let peers in, make sure to have different users for different repos if you need to restrict peers.

# Quick Usage

Assuming you can ssh forward

/etc/hosts
```
192.168.0.2 alpha
192.168.0.3 betasaurus
192.168.0.4 caviar
192.168.0.5 delta
8.8.x.x public
```

Example session from deployment machine that can ssh into all hosts:

```
gtfcr-init alpha
gtfcr-add alpha betasaurus
gtfcr-add alpha caviar
gtfcr-add alpha delta
```

On any host as user `urep`, add a commit to sync
```
cd gitsync
date > testdate
git add .
git commit -m "add testdate"
```
Wait about 60 seconds for the other nodes to pull.

# Can some hosts be read-only?

Kind of. 

Scenario 1
```
For example: We can configure all peers to never
pull from betasaurus because we think he's sketchy.

We can configure them to not pull from certain peers by
removing the betasaurus entry in .gtfc/config which gets
copied to .git/config or even just revoke his key
from authorized_keys2.
```

Scenario 2
```
For example: Maybe we want delta to pause or not be updated.

We can configure a peer to never pull from anyone by
clearing the urep user's crontab.
```

# What can be host specific configurations?

In the `gitsync/.gtfc` folder/repo:

- authorized_keys2.$hostname
- config.$hostname
- crontab.pullall.$hostname
- crontab.pullbare.$hostname
- known_hosts.$hostname

# Long Usage

You'll need to copy .ssh/id_rsa.pub into .ssh/authorized_keys2 before every step

/etc/hosts
```
192.168.0.2 alpha
192.168.0.3 betasaurus
8.8.x.x public
```

1) In a LAN, on `alpha`
```
gtfc init alpha
gtfc add betasaurus
```

copy betasaurus:~/.ssh/id_rsa.pub to alpha:~/.ssh/authorized_keys2

2) On `betasaurus`
```
gtfc clone alpha
```

copy alpha:~/.ssh/id_rsa.pub to betasaurus:~/.ssh/authorized_keys2

3) Back on `alpha`
```
sh gitsync/gitpullall.sh
```

copy alpha:~/.ssh/id_rsa.pub to public:~/.ssh/authorized_keys2

4) On `alpha` or `betasaurus`, to add a commit to sync
```
cd gitsync
date > testdate
git add .
git commit -m "add testdate"
```
Wait about 60 seconds for the other nodes to pull.
