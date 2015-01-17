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
Limitations                                | Bad for large files or binary. Sync based on git commits, not files.

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
ssh urep@alpha      gtfc init  alpha      gitsync

ssh urep@alpha      gtfc add   betasaurus gitsync
ssh urep@betasaurus gtfc clone alpha      gitsync
ssh urep@alpha      sh gitsync/gitpullall.sh -f

ssh urep@alpha      gtfc add   caviar gitsync
ssh urep@caviar     gtfc clone alpha  gitsync
ssh urep@alpha      sh gitsync/gitpullall.sh -f

ssh urep@alpha      gtfc add   delta gitsync
ssh urep@delta      gtfc clone alpha gitsync
ssh urep@alpha      sh gitsync/gitpullall.sh -f
```

On any host, to add a commit to sync
```
cd gitsync
date > testdate
git add .
git commit -m "add testdate"
```
Wait about 60 seconds for the other nodes to pull.

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

4) On `alpha`, to push to `public`
```
gtfc push public
```

5) On `alpha` or `betasaurus`, to add a commit to sync
```
cd gitsync
date > testdate
git add .
git commit -m "add testdate"
```
Wait about 60 seconds for the other nodes to pull.
