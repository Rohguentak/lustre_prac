# lustre_prac
0. ntp설정
1. iml 다운로드 및 서버 추가
2. 러스터 설치
3. lnet 설정 및 시작
4. 러스터 모듈 추가
5. zfs 모듈 추가 
6. zfspool 생성
7. dataset 생성
8. 마운트
9. iml
 
------------------------------------
managed lustre
--------------

yum repolist

yum-config-manager --disable updates
yum install -y createrepo vsftpd
mkdir /repo
mount -t nfs 10.73.10.33:/root/Downloads /repo

tar -xvzf /repo/local_repos.tar.gz -C /var/ftp
createrepo /var/ftp

vi /etc/yum.repos.d/remote.repo

[remote]
name=reomte
baseurl=ftp://10.73.10.10
enabled=1
gpgcheck=0

yum-config-manager --enable remote
yum repolist
yum install -y python2-iml-manager
chroma-config setup admin lustre localhost --no-dbspace-check -v

chroma --username admin --password lustre server add mds1.local --server_profile base_managed_rh7

--------------------------------

lnetctl set discovery [0 | 1]
createrepo -g /path/to/mygroups.xml /srv/my/repo      //그룹list 추가

-------------------------------------------------------------------------------------
monitored zfs
-------------

yum install -y --exclude kernel-debug python2-iml-agent-management kernel-devel-lustre pcs fence-agents fence-agents-virsh lustre-resource-agents lustre-ldiskfs-zfs python2-iml-agent-4.2.0-1.el7

reboot

lnet config
config ntp

pcs cluster auth 10.128.0.11 -u hacluster -p ********************: 0
10.128.0.11: Authorized

pcs cluster node add 10.128.0.11: 0
Disabling SBD service...
10.128.0.11: sbd disabled
Sending remote node configuration files to '10.128.0.11'
10.128.0.11: successful distribution of the file 'pacemaker_remote authkey'
10.128.0.12: Corosync updated
Setting up corosync...
10.128.0.11: Succeeded
Synchronizing pcsd certificates on nodes 10.128.0.11...
10.128.0.11: Success
Restarting pcsd on the nodes in order to reload the certificates...
10.128.0.11: Success

config corosync pcs cluster auth 10.128.0.12 -u hacluster -p ********************: 0
10.128.0.12: Authorized

pcs cluster setup --name lustre-ha-cluster --force 10.128.0.12 --transport udp --rrpmode passive --addr0 10.128.0.0 --mcast0 226.94.0.1 --mcastport0 42227 --addr1 10.73.10.0 --mcast1 226.94.1.1 --mcastport1 42227 --token 17000 --fail_recv_const 10
Destroying cluster on nodes: 10.128.0.12...
10.128.0.12: Stopping Cluster (pacemaker)...
10.128.0.12: Successfully destroyed cluster

Sending 'pacemaker_remote authkey' to '10.128.0.12'
10.128.0.12: successful distribution of the file 'pacemaker_remote authkey'
Sending cluster config files to the nodes...
10.128.0.12: Succeeded

Synchronizing pcsd certificates on nodes 10.128.0.12...
10.128.0.12: Success
Restarting pcsd on the nodes in order to reload the certificates...
10.128.0.12: Success

stonith resource 만 등록되어있음

디스크 포맷

mkfs.lustre --mgs --failnode=10.73.20.12@tcp0 --backfstype=zfs --mkfsoptions="mountpoint=none" mgtpool/MGS

mkfs.lustre --mdt --mgsnode=10.73.20.11@tcp0 --mgsnode=10.73.20.12@tcp0 --failnode=10.73.20.11@tcp0 --index=0 --backfstype=zfs --fsname=hello --mkfsoptions="mountpoint=none" mdtpool/hello-MDT0000

mkfs.lustre --ost --mgsnode=10.73.20.11@tcp0 --mgsnode=10.73.20.12@tcp0 --failnode=10.73.20.22@tcp0 --index=0 --backfstype=zfs --fsname=hello --mkfsoptions="mountpoint=none" ostpool1/hello-OST0000

mkfs.lustre --ost --mgsnode=10.73.20.11@tcp0 --mgsnode=10.73.20.12@tcp0 --failnode=10.73.20.21@tcp0 --index=1 --backfstype=zfs --fsname=hello --mkfsoptions="mountpoint=none" ostpool2/hello-OST0001




