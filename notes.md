---
layout: page
title: Notes
---

* TOC will be output here
{:toc}

***

General
-----------------------------

### Add SSH key to LDAP user using ldapmodify

john.ldif
```
dn: uid=john,ou=user,dc=example,dc=com
changetype: modify
add: objectclass
objectclass: ldapPublicKey
-
add: sshPublicKey
sshPublicKey: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDQOZcZYLg83R1owBWQbW2ieEp8cwNbAkju1ETm7M/NG2Bfgr2XwoZVo22Drsx3QgZzp0w5m/TbkOVrQM7gRngacl0lh0J/r0hsuPxi/pW36Zt2GG8nvAN9WfllMsPqp/XeaKo1OqMN98MQdqYH2HdeWV7aqnbo5jXcWL0zqUrEum1sAZbrDrwdDxfxrK4TDLW14GdSkesZht9r963eplxTMSTFLJP5HDsqWbaH3+8RdCiUA8tVHmqDZD8wYkmAYAzksPmAskewVo08TG9j0/Id6Oa+6C3rVJDBbgHOwQUkmvO90v4HkkSheMbymtcgsU6mOyEs3z3f7k6KMO8yEm4p
```

Then run:
```shell
ldapmodify -D "cn=manager,dc=example,dc=com" -W -f john.ldif

Enter LDAP Password:

modifying entry "uid=john,ou=user,dc=example,dc=com"
```

### ldapsearch CLI

```shell
ldapsearch -W -D 'cn=manager,dc=example,dc=com' -H ldaps://ldap.example.com
```

### Rename folder naming to sequence


I want to rename this:

mt-worker-1
mt-worker-2
mt-worker-3

To

mt-worker-01
mt-worker-02
mt-worker-03


```shell
rename -v mt-worker- mt-worker-0 mt-worker-?
```

### Sort CSV by Column

Sort by 4th column
```shell
sort -t, -k4 -n memory.csv
```

### Run a script as apache user
```shell
su - apache -s /bin/bash
php script.php
```

### ACL
```shell
setfacl -Rd -m u:<user>:rwX <dir>
```

### Redis resque list all queues
```shell
redis-cli SMEMBERS resque:queues
```
### Redis resque check how many items in queue
```shell
redis-cli LLEN resque:queue:<queue_name>
```

### Find git repository on gitlab-ci agent
```shell
cd /home/gitlab-runner/builds
find . -maxdepth 5 -type d -name mcb -ls
```

### convert timestamp (epoch) to date
```shell
date --date @1496183649
```

### Inspect network for specific packets
```shell
ngrep -d any port 8125 <filter to grep for>
```

### Show UDP statistics
```shell
netstat -s --udp
```

### Citus - kill running query
```shell
SELECT pg_terminate_backend(<pid>);
```

### Find inodes usage on current directory
```shell
find . -xdev -type f | cut -d "/" -f 2 | sort | uniq -c | sort -n
```
Refrence: https://stackoverflow.com/a/9387415/3238982

Sed
-----------------------------

### Search and replace
```shell
sed -i 's/DAEMON=\"\/usr\/bin\/php/DAEMON=\"$DOCKER_CMD_HEADER \/usr\/bin\/php/g' $(grep -l "DAEMON=\"/usr/bin/php" */rc.main)
```
```shell
sed -i 's/\/usr\/bin\/php ..\/..\//\/usr\/bin\/php /g' $(grep -l "/usr/bin/php ../../" */rc.main)
```

### Insert multiple lines in line number 8
```shell
sed -i '8iif [ "$TARGET" = "trap" ]; then\n  TARGET=_trap\nfi\n' my-*/rc.main
```
### Replace user apache with docker command in crontab file
```shell
sed -i 's/apache /root docker run .../g' crontab
```

### Only search and replace for specific lines
```/tmp/file
four five six
one
seve eight nine
one two three
one
ten eleven twelve
one
```
```
sed -e '/two/s/one/replaced/g' /tmp/file
```
Ref: https://www.golinuxhub.com/2017/09/sed-perform-search-and-replace-only-on.html

Strace
-----------------------------
```shell
strace -s2000 -f -e '!lstat,munmap,mmap,fstat,clock_gettime,gettimeofday,ioctl,stat' -ttt -p 71503
```

### ssh only using password
```shell
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no example.com
```
Zenoss
-----------------------------

### module device CLI
```shell
zenmodeler run -d <server_hostname> -v10
```

### zendmd get memory cache value
```shell
zendmd
>>> z = [x for x in devices.getSubDevices() if x.id == '<server_name>']
>>> d.getRRDValue("memCached_memCached")
```

Sysdig
-----------------------------

### run sysdig on rancher os
To run sysdig on rancheros, you need kernel-header to be installed
```shell
sudo ros service enable kernel-headers

sudo ros service up -d kernel-headers

docker run -it --rm --name=sysdig --privileged=true \
           --volume=/var/run/docker.sock:/host/var/run/docker.sock \
           --volume=/dev:/host/dev \
           --volume=/proc:/host/proc:ro \
           --volume=/boot:/host/boot:ro \
           --volume=/lib/modules:/host/lib/modules:ro \
           --volume=/usr:/host/usr:ro \
           sysdig/sysdig
```

### Apply filter to csysdig in dig mode
Click on Filter section on top of the csysdig output and added your event type filters like below,
bellow example elemenates `gettimeofday`, `epoll_wait` and `switch` event types.
```shell
(evt.type!=switch) and proc.pid=22973 and evt.type != gettimeofday and evt.type != epoll_wait and evt.type != switch
```
Full list of filters that can apply https://github.com/draios/sysdig/wiki/Sysdig-User-Guide#filtering

Route53 CLI
-----------------------------

### List all domains
```shell
cli53 list | grep Name | awk '{print $2}
```

### Dump all records from all domains
```shell
cli53 list | grep 'Name:' | awk -F\" '{print$2}' | xargs -n1 bin/cli53 export -f | tee fulldump
```

 Unbound
-----------------------------

### Clear DNS cache
```shell
unbound-control flush_zone example.org.
```

MySQL
-----------------------------

### dump one row
```shell
mysqldump -t -u root -p mydb account --where="accountid=8242250"
```

### Delete any duplicate in rows and newer than date
```shell
delete n1 FROM blocklist n1, blocklist n2 WHERE n1.listid > n2.listid AND n1.MSISDN = n2.MSISDN AND n1.CreateDate > "2016-11-23 00:00:00";
```

### one liner command to skip duplicate entry in mysql replication
```shell
while `sleep 2`; do mysql -e "show slave status;" | grep "Duplicate entry" && mysql -e "stop slave; SET GLOBAL sql_slave_skip_counter = 1; start slave;" ; done
```

### Sync table with percona tools
```shell
pt-table-checksum --replicate=percona.checksums --tables <TABLE>  --no-check-replication-filters --no-check-binlog-format --recursion-method=dsn=D=percona,t=dsns h=localhost,u=<USER>,p=<PASSWORD>
pt-table-sync --print --databases <DB> --tables <TABLE> --check-child-tables --replicate=percona.checksums --sync-to-master h=localhost,u=<USER>,p=<PASSWORD> > /tmp/<DB>-<TABLE>.sql
```

AWK
-----------------------------

### Failed HTTP request count - haproxy
```shell
awk '{if ($11 >= 500) print }' /var/log/haproxy.log | grep -P ":07:(2|3)\d:(\d+).(\d+)" | awk '{ if ($NF >= "HTTP/1.0") print $(NF-1);else print $NF}' | awk -F"?" '{print $1}' | sort | uniq -c
```

### Find requests slower than 3 seconds - haproxy
```shell
tail -f /var/log/haproxy.log | awk -F '[/ ]' '{if ($17>=3000) print}'
```

### Grep slow response from server side - haproxy
```shell
tail -f /var/log/haproxy.log | awk -F '[/ ]' '{if ($16>=3000) print}'
```

### Print column before last one
```
awk '{print $(NF-1)}'
```

SaltStack
-----------------------------

### Refresh gitfs states
```shell
salt-run fileserver.update
```

### Refresh gitfs pillar
```shell
salt-run git_pillar.update
salt '*' saltutil.refresh_pillar
```

Apache (httpd)
-----------------------------

### Allow permission for certain IPs
```
    <Location /uploads>
        Order Deny,Allow
        Deny from all
        Allow from localhost
        Allow from x.x.x.x
        Allow from y.y.y.y
        Allow from z.z.z.z
    </Location>
```

Go language
-----------------------------

### Compile binary for different OS
List of Operating systems and architecture in
https://golang.org/doc/install/source#environment
```shell
env GOOS=linux GOARCH=amd64 go get -v github.com/rubenv/sql-migrate/...
```

### Force return error
```
import "errors"

func failFunc() error {
    return errors.New("Error message")
}
```
Ref: https://stackoverflow.com/a/36311290

### Retry
Ref: https://godoc.org/github.com/avast/retry-go

Git
-----------------------------

### Ability to manage multiple github accounts

* Generate ssh key and name it for example `personal_id_rds`
* Edit `~/.ssh/config` and add the following
```
Host github.personal
   HostName github.personal
   User git
   IdentityFile ~/.ssh/personal_id_rsa
```
* Clone your personal project and edit it's config `.git/config`
```
[remote "origin"]
        url = git@github.personal:mhdali/mhdali.github.io.git
```
NOTE: we changed the hostname from `github.com` to `github.personal`.

### Add upstream branch
```
git remote add upstream git@github.com:...
```

### ignore directories in git grep
```
git grep foobar -- './*' ':!*.java'
```
Ref: https://stackoverflow.com/a/30084612

VIM
-----------------------------

### Interactive search and replace
```
:%s/old/new/gc
```


## Terraform
### taint resource
if you have resource with full name as this: 
module.tst_cluster.module.foo.tfe_variable.role_arn
```
tf taint -module=tst_cluster.foo tfe_variable.key_secret
```

## KMS
### Find out which key encrypt a text
```
aws kms decrypt --ciphertext-blob fileb://<(echo '<ENCRYPTED_BLOB>' | base64 -D) --output text
```
Results will be something like this `key ARN` `ecnrypted text`:
```
arn:aws:kms:<REGION>:<ACCOUNT_ID>:key/<KEY_ID>	<ENCRYPTED_TEXT>
```

BASH
-----------------------------
### Remove an item from array
```
array=(aardvark baboon "clouded leopard" dolphin)
while (( ${#array[@]} ))
do
  echo "Animal: ${array[0]}"
  array=( "${array[@]:1}" )
done
```
OR
```
array=(aardvark baboon "clouded leopard" dolphin)
set -- "${array[@]}"
while (( $# ))
do
  echo "Animal: $1"
  shift
done
```
Ref: https://stackoverflow.com/questions/50846539/shift-in-bash-but-on-arrays-instead-of-arguments

### Check if array is empty
```
if [ -z "${ARRAY:-}" ]; then
  error "Array is empty"
fi
```
define default value in case you're using `-u` option

Ref: https://serverfault.com/a/700936

### Get uniqe value of array
```
ARR=("foo" "foo" "bar")

echo "${ARR[@]}" | tr ' ' '\n' | uniq | tr '\n' ' '
```
Ref: https://stackoverflow.com/a/13648438

### Append an item to array
```
ARR+=("foo")
ARR+=("bar")
```
Ref: https://stackoverflow.com/a/1951523

### Source directory of the script
```
dirname "${BASH_SOURCE[0]}"
```
Ref: https://stackoverflow.com/a/246128

### Reference a function from script to another
```second.sh
function foo {
  echo "foo"
}
```
```first.sh
source ./second.sh
foo
```

Ref: https://stackoverflow.com/a/10823213

### Get older than 5 folders/files
```
ls -tr | head -n -5
```
OR
```
ls -tr | head -$(($(ls | wc -l)-5))
```

Ref: https://stackoverflow.com/a/18127758
