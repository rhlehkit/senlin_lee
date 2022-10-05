========================
Lee Senlin
========================

Senlin에 대한 분석 시작 

10월 5일 

대략적인 구조 파악 

Senlin의 경우 oslo_* 라는 프레임워크(?) 를 상속 받아 구현되어 있음 

다른 친구들(nova, 뉴트론 등등)도 해당 프레임워크를 상속 받아 구현되어 있는데 그 이유는 2가지로 추측 

1. 각 모듈에 대한 코드 관리(?) 구조에 대해 공통으로 가져가기 위해

2. 각 모듈(?) 간에 통신의 경우 RPC로 구현되어 있음 이에 대한 지원을 위해 ?


ref : 
Oslo : https://wiki.openstack.org/wiki/Oslo
rpc : 


Database 관련

블리자드의 Senlin 적용기에 따르면 특히 많은 클러스터가 운영 중일 때 데이터베이스 효율성이 떨어졌고 액션 리스트, 고객 리스트 작업에서 지연이 나타났다

라고 한다. 그 이유를 찾아보면



원문보기:
https://www.ciokorea.com/tags/662/%EC%98%A4%ED%94%88%EC%8A%A4%ED%83%9D/122204#csidx365d0a2ce03c92b9a43ce1eba536853 




Senlin
======

--------
Overview
--------

Senlin is a clustering service for OpenStack clouds. It creates and operates
clusters of homogeneous objects exposed by other OpenStack services. The goal
is to make the orchestration of collections of similar objects easier.

Senlin provides RESTful APIs to users so that they can associate various
policies to a cluster.  Sample policies include placement policy, load
balancing policy, health policy, scaling policy, update policy and so on.

Senlin is designed to be capable of managing different types of objects. An
object's lifecycle is managed using profile type implementations, which are
themselves plugins.

---------
For Users
---------

If you want to install Senlin for a try out, please refer to the documents
under the ``doc/source/user/`` subdirectory. User guide online link:
https://docs.openstack.org/senlin/latest/#user-references

--------------
For Developers
--------------

There are many ways to help improve the software, for example, filing a bug,
submitting or reviewing a patch, writing or reviewing some documents. There
are documents under the ``doc/source/contributor`` subdirectory. Developer
guide online link: https://docs.openstack.org/senlin/latest/#developer-s-guide

---------
Resources
---------

Launchpad Projects
------------------
- Server: https://launchpad.net/senlin
- Client: https://launchpad.net/python-senlinclient
- Dashboard: https://launchpad.net/senlin-dashboard
- Tempest Plugin: https://launchpad.net/senlin-tempest-plugin

Code Repository
---------------
- Server: https://opendev.org/openstack/senlin
- Client: https://opendev.org/openstack/python-senlinclient
- Dashboard: https://opendev.org/openstack/senlin-dashboard
- Tempest Plugin: https://opendev.org/openstack/senlin-tempest-plugin

Blueprints
----------
- Blueprints: https://blueprints.launchpad.net/senlin

Bug Tracking
------------
- Server Bugs: https://bugs.launchpad.net/senlin
- Client Bugs: https://bugs.launchpad.net/python-senlinclient
- Dashboard Bugs: https://bugs.launchpad.net/senlin-dashboard
- Tempest Plugin Bugs: https://bugs.launchpad.net/senlin-tempest-plugin

Weekly Meetings
---------------
- Schedule: every Tuesday at 1300 UTC, on #openstack-meeting channel
- Agenda: https://wiki.openstack.org/wiki/Meetings/SenlinAgenda
- Archive: http://eavesdrop.openstack.org/meetings/senlin/2015/

IRC
---
IRC Channel: #senlin on `OFTC`_.

Mailinglist
-----------
Project use http://lists.openstack.org/cgi-bin/mailman/listinfo/openstack-discuss
as the mailinglist. Please use tag ``[Senlin]`` in the subject for new
threads.


.. _OFTC: https://oftc.net/

Release notes
------------------
- Release notes: https://docs.openstack.org/releasenotes/senlin/
