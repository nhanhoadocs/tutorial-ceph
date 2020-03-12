Below steps are taken from redhat documentation:

Follow the below procedure for Shutting down the Ceph Cluster:
1.    Stop the clients from using the RBD images/Rados Gateway on this
cluster or any other clients.
2.    The cluster must be in healthy state before proceeding.
3.    Set the noout, norecover, norebalance, nobackfill, nodown and pause flags
```
#ceph osd set noout
#ceph osd set norecover
#ceph osd set norebalance
#ceph osd set nobackfill
#ceph osd set nodown
#ceph osd set pause
```
4.    Shutdown osd nodes one by one
5.    Shutdown monitor nodes one by one
6.    Shutdown admin node

For Bringing up follow the below order:
1.    Power on the admin node
2.    Power on the monitor nodes
3.    Power on the osd nodes
4.    Wait for all the nodes to come up , Verify all the services are
up and the connectivity is fine between the nodes.
5.    Unset all the noout,norecover,noreblance, nobackfill, nodown and
pause flags.
```
#ceph osd unset noout
#ceph osd unset norecover
#ceph osd unset norebalance
#ceph osd unset nobackfill
#ceph osd unset nodown
#ceph osd unset pause
```
6.    Check and verify the cluster is in healthy state, Verify all the
clients are able to access the cluster.
