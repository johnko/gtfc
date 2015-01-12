# gtfc
git for cluster, naive synchronization (pull only) over ssh every minute (with crontab)

# Quick Usage

Assuming you can ssh forward

/etc/hosts
```
192.168.0.2 alpha
192.168.0.3 betasaurus
8.8.x.x public
```

1. In a LAN, on `alpha`
```
ssh urep@alpha gtfc init alpha
```

2. On `betasaurus`
```
ssh -A urep@betasaurus gtfc clone alpha
```

3. Back on `alpha`
```
sh gitsync/gitpullall.sh
```

4. On `alpha`, to push to `public`
```
ssh -A urep@alpha gtfc push public
```


# Long Usage

You'll need to copy .ssh/id_rsa.pub into .ssh/authorized_keys2 before every step

/etc/hosts
```
192.168.0.2 alpha
192.168.0.3 betasaurus
8.8.x.x public
```

1. In a LAN, on `alpha`
```
gtfc init alpha
gtfc add betasaurus
```

copy betasaurus:~/.ssh/id_rsa.pub to alpha:~/.ssh/authorized_keys2

2. On `betasaurus`
```
gtfc clone alpha
```

copy alpha:~/.ssh/id_rsa.pub to betasaurus:~/.ssh/authorized_keys2

3. Back on `alpha`
```
sh gitsync/gitpullall.sh
```

copy alpha:~/.ssh/id_rsa.pub to public:~/.ssh/authorized_keys2

4. On `alpha`, to push to `public`
```
gtfc push public
```
