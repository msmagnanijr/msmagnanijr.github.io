---
layout: post
title: Create your own OpenShift Plugin
tags: [plugin, openshift, cloud, troubleshooting, etcd]
comments: true
---

##  Writing Plug-ins

A very interesting feature was made available as a tech preview from OpenShift 3.7. This feature allows the OpenShift cluster administrator to create custom plugins to improve productivity in administration/troubleshooting.

I will not go into detail because we have clear [documentation](https://docs.openshift.com/container-platform/3.9/cli_reference/extend_cli.html) of what is required but on the other hand I will describe below some steps for you to write your first plugin.

To make it easier to manage versions and clones in other environments, create a repository in github such as [openshift-etcd-plugin](https://github.com/msmagnanijr/openshift-etcd-plugin).

In the openshift-etcd-plugin.git directory, create the etcd.sh and plugin.yaml files.

The plugin.yml file is the central point of calls for scripts, binaries, etc.

Example:

{% highlight yaml %}
name: "etcd"
shortDesc: "Plugin for check etcd info from OpenShift."
command: "./etcd.sh"
flags:
  - name: "output"
    desc: "Output format of data "
{% endhighlight %}

The etcd.sh file will take the actions needed to get this information:

{% highlight bash %}
#!/bin/bash
exec {BASH_XTRACEFD}>>$DEST/etcd.log
set -x
openshift version
source /etc/etcd/etcd.conf
export ETCDCTL_API=3
ETCD_ALL_ENDPOINTS=` etcdctl  --cert=$ETCD_PEER_CERT_FILE --key=$ETCD_PEER_KEY_FILE --cacert=$ETCD_TRUSTED_CA_FILE --endpoints=$ETCD_LISTEN_CLIENT_URLS --write-out=fields   member list | awk '/ClientURL/{printf "%s%s",sep,$3; sep=","}'`
etcdctl  --cert=$ETCD_PEER_CERT_FILE --key=$ETCD_PEER_KEY_FILE --cacert=$ETCD_TRUSTED_CA_FILE --endpoints=$ETCD_LISTEN_CLIENT_URLS --write-out=table  member list
etcdctl  --cert=$ETCD_PEER_CERT_FILE --key=$ETCD_PEER_KEY_FILE --cacert=$ETCD_TRUSTED_CA_FILE --endpoints=$ETCD_ALL_ENDPOINTS  --write-out=table endpoint status
etcdctl  --cert=$ETCD_PEER_CERT_FILE --key=$ETCD_PEER_KEY_FILE --cacert=$ETCD_TRUSTED_CA_FILE --endpoints=$ETCD_ALL_ENDPOINTS endpoint health
{% endhighlight %}

Push the files to the repository in github.

Now in your cluster openshift clone repository into your ~/.kube/plugins directory:

{% highlight bash %}
[root@master-0]# git clone https://github.com/msmagnanijr/openshift-etcd-plugin ~/.kube/plugins/openshift-etcd-plugin
{% endhighlight %}

Verify that it is working:

{% highlight bash %}
[root@master-0]# oc plugin
Runs a command-line plugin. 

Plugins are subcommands that are not part of the major command-line distribution and can even be provided by
third-parties. Please refer to the documentation and examples for more information about how to install and write your
own plugins.

Usage:
  oc plugin NAME [options]

Available Commands:
  etcd        Plugin for gathering etcd info from OpenShift.
{% endhighlight %}

Now simply run:
{% highlight bash %}
[root@master-0 openshift-etcd-plugin]# oc plugin etcd
+ openshift version
openshift v3.9.25
kubernetes v1.9.1+a0ce1bc657
etcd 3.2.16
+ source /etc/etcd/etcd.conf
++ ETCD_NAME=master-0.mmagnani.example.com
++ ETCD_LISTEN_PEER_URLS=https://10.10.30.45:2380
++ ETCD_DATA_DIR=/var/lib/etcd/
++ ETCD_HEARTBEAT_INTERVAL=500
++ ETCD_ELECTION_TIMEOUT=2500
++ ETCD_LISTEN_CLIENT_URLS=https://10.10.30.45:2379
++ ETCD_INITIAL_ADVERTISE_PEER_URLS=https://10.10.30.45:2380
++ ETCD_INITIAL_CLUSTER=master-0.mmagnani.example.com=https://10.10.30.45:2380
++ ETCD_INITIAL_CLUSTER_STATE=new
++ ETCD_INITIAL_CLUSTER_TOKEN=etcd-cluster-1
++ ETCD_ADVERTISE_CLIENT_URLS=https://10.10.30.45:2379
++ ETCD_QUOTA_BACKEND_BYTES=4294967296
++ ETCD_TRUSTED_CA_FILE=/etc/etcd/ca.crt
++ ETCD_CLIENT_CERT_AUTH=true
++ ETCD_CERT_FILE=/etc/etcd/server.crt
++ ETCD_KEY_FILE=/etc/etcd/server.key
++ ETCD_PEER_TRUSTED_CA_FILE=/etc/etcd/ca.crt
++ ETCD_PEER_CLIENT_CERT_AUTH=true
++ ETCD_PEER_CERT_FILE=/etc/etcd/peer.crt
++ ETCD_PEER_KEY_FILE=/etc/etcd/peer.key
++ ETCD_DEBUG=False
+ export ETCDCTL_API=3
+ ETCDCTL_API=3
++ etcdctl --cert=/etc/etcd/peer.crt --key=/etc/etcd/peer.key --cacert=/etc/etcd/ca.crt --endpoints=https://10.10.30.45:2379 --write-out=fields member list
++ awk '/ClientURL/{printf "%s%s",sep,$3; sep=","}'
+ ETCD_ALL_ENDPOINTS='"https://10.10.30.45:2379"'
+ etcdctl --cert=/etc/etcd/peer.crt --key=/etc/etcd/peer.key --cacert=/etc/etcd/ca.crt --endpoints=https://10.10.30.45:2379 --write-out=table member list
+------------------+---------+------------------------------------------------+---------------------------+---------------------------+
|        ID        | STATUS  |                      NAME                      |        PEER ADDRS         |       CLIENT ADDRS        |
+------------------+---------+------------------------------------------------+---------------------------+---------------------------+
| 10639905e92fe697 | started | master-0.mmagnani.example.com | https://10.10.30.45:2380 | https://10.10.30.45:2379 |
+------------------+---------+------------------------------------------------+---------------------------+---------------------------+
+ etcdctl --cert=/etc/etcd/peer.crt --key=/etc/etcd/peer.key --cacert=/etc/etcd/ca.crt '--endpoints="https://10.10.30.45:2379"' --write-out=table endpoint status
+---------------------------+------------------+---------+---------+-----------+-----------+------------+
|         ENDPOINT          |        ID        | VERSION | DB SIZE | IS LEADER | RAFT TERM | RAFT INDEX |
+---------------------------+------------------+---------+---------+-----------+-----------+------------+
| https://10.10.30.45:2379 | 10639905e92fe697 |  3.2.15 |   14 MB |      true |         2 |    2608889 |
+---------------------------+------------------+---------+---------+-----------+-----------+------------+
+ etcdctl --cert=/etc/etcd/peer.crt --key=/etc/etcd/peer.key --cacert=/etc/etcd/ca.crt '--endpoints="https://10.10.30.45:2379"' endpoint health
https://10.10.30.45:2379 is healthy: successfully committed proposal: took = 849.061µs
{% endhighlight %}

It seems like a very interesting feature! The example I presented was quite simple.

You can find an excellent example of use at [OpenShift SOS Plugin]( https://github.com/bostrt/openshift-sos-plugin)

See you!
