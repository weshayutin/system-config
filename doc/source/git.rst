:title: Git

.. _git:

Git
########

The web frontend cgit is running on git.openstack.org.

At a Glance
===========

:Hosts:
  * https://git.openstack.org
  * git*.openstack.org
:Puppet:
  * https://git.openstack.org/cgit/openstack-infra/puppet-cgit/tree/
  * :cgit_file:`modules/openstack_project/manifests/git.pp`
:Configuration:
  * :cgit_file:`modules/openstack_project/files/git/cgitrc`
:Projects:
  * http://git.zx2c4.com/cgit/
:Bugs:
  * https://storyboard.openstack.org/#!/project/748
  * http://lists.zx2c4.com/mailman/listinfo/cgit

Overview
========

The OpenStack git repositories are hosted on a pool of CentOS servers with the
EPEL repository that includes the cgit package. They are served up via https
using cgit and via git:// by git-daemon behind HAProxy which handles load
balancing across the nodes.

SELinux is enabled on these nodes and requires restorecon to be run against
/var/lib/git to set the appropriate SELinux security context. SELinux rules are
also in place to allow for Apache to run on a non-standard port so it can sit
behind HAProxy. This is all handled by puppet.

In order to mitigate potential problems with HTTP(S) responses, HAProxy is
configured using the source balance method so that every request from a single
host will be served by one backend node unless nodes are added or removed.

The jeepyb script create-cgitrepos runs against projects.yaml to generate the
/etc/cgitrepos file listing all the git repositories. The git repositories are
synced to all the nodes from the Gerrit server.

Backend Maintenance
===================

To temporarily remove a git backend from the HAProxy load balancer,
you can put it in "maintenance" mode.  This can be done interactively
on the HAProxy host.  Note that long-term changes to the topology
should be made via configuration management.  These commands must be
run as root.

To see the current status of all servers::

  echo "show stat" | socat /var/lib/haproxy/stats stdio

To disable a server (eg, git08)::

  echo "disable server balance_git_daemon/git08.openstack.org" | socat /var/lib/haproxy/stats stdio
  echo "disable server balance_git_http/git08.openstack.org" | socat /var/lib/haproxy/stats stdio
  echo "disable server balance_git_https/git08.openstack.org" | socat /var/lib/haproxy/stats stdio

To re-enable a server::

  echo "enable server balance_git_daemon/git08.openstack.org" | socat /var/lib/haproxy/stats stdio
  echo "enable server balance_git_http/git08.openstack.org" | socat /var/lib/haproxy/stats stdio
  echo "enable server balance_git_https/git08.openstack.org" | socat /var/lib/haproxy/stats stdio

To run these commands and others interactively, issue the prompt
command to haproxy::

  socat readline /var/lib/haproxy/stats
  prompt
