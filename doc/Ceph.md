### Common

* `ceph -w` - show running cluster ops (recovery etc)


### Key management

Note that create options cant modify existing permissions, you have to use `auth caps` for it

* `ceph auth get-or-create client.nagios mon 'allow r'` - create nagios user with rights to monitor and return his key
* `ceph auth get-or-create client.hypervisor mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=volumes'` - create user with enough rights for libvirt to use storage on 'volumes'
* `ceph-authtool -C -n client.hypervisor --gen-key keyring` - name is of type.user format so clients are `client.name`, osds are `osd.0` etc
* `ceph auth list` - list auth
* `ceph auth caps client.hypervisor osd 'allow rw pool=volumes'` - modify permissions for user. `''` for none. You **have to** specify all (mon+osd, or mon+osd+mds), if you dont it will zero out other permissions. So if you change osd perms, copy-paste mon permissions too
* `ceph auth import -i /etc/ceph/client.nagios.keyring` - import a keyring

### Pool management

* `ceph osd pool create volumes 512` - create pool `volumes` with `512` placement groups
* `ceph osd pool set {pool-name} pg_num {pg_num}` - change placement group count (disruptive). Remember to also change pgp_num after that
* `ceph osd pool set {pool-name} pgp_num {pgp_num}` - change placement group for placement count (disruptive). Remember to change pg_num before that


### Volume management

* `rbd create mypool/myimage --size 102400` - create 100GB image. THIS IS IN MEGABYTES and it doesnt support usual k/M/G/T suffixes
!


### OSD

* `ceph-osd -i 11 --mkjournal` - initialize journal
* `ceph-osd -i 11 --flush-journal` - flush journal (stop osd service first!)
* `ceph-osd -i 11 --mkfs --mkkey` - create fs and key for osd
* `ceph auth add osd.8  osd 'allow *' mon 'allow rwx' -i /var/lib/ceph/osd/ceph-8/keyring` - add key for osd
* `ceph osd create 2e6ee69a-5477-40b2-9eea-c675ef8ca2a3` - create osd from blkuid
#### Removing OSD

    ceph osd crush reweight osd.{osd-num} 0 # wait for rebalance
    ceph osd out {osd-num}
    ceph osd crush remove osd.{osd-num}
    ceph auth del osd.{osd-num}
    ceph osd rm {osd-num}

#### osd recovery

* `ceph osd set noin` - do not add new ones automatically
* `ceph osd set nobackfill` - do not start backfill
* `ceph osd set norecover` - do not start recovery

### Rados GW

#### Common ops

* `radosgw-admin user create --uid="username" --display-name="long username"`
* `radosgw-admin user rm --uid=username`
* `radosgw-admin bucket list`

#### Create pools
`for a in .rgw.root .rgw.control .rgw.gc .rgw.buckets .rgw.buckets.index .rgw.buckets.extra .log .intent-log .usage .users .users.email .users.swift .users.uid ; do ceph osd pool create $a 16 16 ; done`

will create small (16 PG) pools for RADOS; you can tune up but not down (AFAIK, might be fixed) so it is better to start small

### Gotchas

* `ceph osd crush tunables optimal` - run on new cluster to use optimal profile instead of legacy; will cause rebalance
