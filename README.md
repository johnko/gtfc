# gtfc
git for cluster sync, naive synchronization (pull only) over ssh every minute (with crontab)

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
ssh urep@alpha gtfc init alpha
ssh -A urep@betasaurus gtfc clone alpha ; ssh urep@alpha sh gitsync/gitpullall.sh -f
ssh -A urep@caviar gtfc clone alpha ; ssh urep@alpha sh gitsync/gitpullall.sh -f
ssh -A urep@delta gtfc clone alpha ; ssh urep@alpha sh gitsync/gitpullall.sh -f
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
