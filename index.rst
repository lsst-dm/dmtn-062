..
  Technote content.

  See https://developer.lsst.io/docs/rst_styleguide.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. Add content below. Do not include the document title.

.. note::

   **This technote is not yet published.**

   A look at using OpenShift in LSST

.. Add content here.




Overview
--------

This document describes our experience with OpenShift, and assumes some understanding of Kubernetes.  The OpenShift terms apply to both OpenShift Origin and OpenShift Enterprise.   It includes a discussion of the requirements for the LSST LDF.

OpenShift and Kubernetes
^^^^^^^^^^^^^^^^^^^^^^^^

Redhat's OpenShift is a platform as a service (PAAS) that runs over the top of Kubernetes, which is a container as a service (CAAS) platform.  OpenShift Origin is the community version of this software, and OpenShift Enterprise is the commercially supported version.

Many of the features that began in OpenShift have migrated into Kubernetes.  Roles, authentication and authorization, siloing of users and resources based on namespaces (OpenShift projects), where all first OpenShift features.    These features in particular are some we would like to leverage for the LSST LDF Kubernetes cluster.

Installation Experience 
^^^^^^^^^^^^^^^^^^^^^^^

The initial attraction of OpenShift Origin was that it might be a cleaner alternative to installing straight Kubernetes.  In looking at Kubernetes we ran into issues with very frequent releases, sometimes within days of each other.  It turned out that the lack of QA in releases became a problem several times.  For example, we ran into issues where installing according to the instructions failed.   

We were hopeful that RedHat's OpenShift Origin would result in an easier install and more stable environment that didn't require constant updating. This community is also quite active and the updates are frequent.  However, OpenShift Origin also seems to have issues with release engineering.  We've run into a couple of problems in some point releases, where even simple installation bugs could have been caught had a simple regression test been done.  

The installation documentation from the website is somewhat misleading.  The version of the install instructions first say that either Atomic or RedHat Linux can be used.  If the instructions for Linux are followed, a single node OpenShift instance will be created.   When attempting to add more nodes to the cluster, the documentation then stated that the preferred method of installing really is to use Atomic and Ansible, and offers some instructions on how to do that.  All this in an of itself isn't a big problem, but it did cause some confusion.  There have also been some pretty big shifts in how the command line interfaces to OpenShift have changed, and made some of the online tutorials and other available online material obsolete.

A further search into more recent instructions from a third party yielded information that was close to what we needed to install everything properly, using RedHat and Ansible. After some modifications for our OpenStack cluster, we wrote our own set of instructions and several scripts to facilitate the install.  They are available at:

        https://github.com/lsst-dm/openshift


This procedure allowed us to install the manager and worker nodes on OpenStack successfully.   

The user experience of OpenShift Origin is somewhat similar to Kubernetes, but since this is deployed to a smaller audience, finding solutions to issues is somewhat more difficult since some errors are sometime a bit opaque and there is less information online.



OpenShift Projects
^^^^^^^^^^^^^^^^^^

An OpenShift "project" is an extension of the Kubernetes' "namespace".  A namespace is used to partition resources in a cluster.   A project builds on this concept allowing for user and resource isolation.  Users, containers, and other resources in one project cannot interact with another project, unless authorized by the OpenShift administrator to do so. Each project can have its own set of administrators, resources (pods, services, etc), policies for what users can and can't do with resources, and quotas.  Note that "administrator" references only to the OpenShift resources and does not require root access on the machines where OpenShift is hosted.

This functionality has been migrated into Kubernetes from OpenShift.

OpenShift Authentication and Authorization
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As in Kubernetes, all interaction with OpenShift is done via calls to the REST API of the central manager(s).   This API is used to manage all applications, users, policies and nodes in the cluster itself. 
 In order to access the OpenShift API, users are first authenticated. This identifies the user and their role in the project they're accessing. All OpenShift API calls are made with an access token or an X.509 certificate (depending on how the API is invoked). The user's role is then used to check internally whether requests they are making are authorized.  
Authorization can be given to individuals or to groups.  A user can also make requests on behalf of other users, as long as the requestor has those rights.
Authorization policies determine what a user can do within a project.    There are two levels of authorization, cluster-wide and project-wide.  Cluster-wide authorizations describe roles users have over the cluster in general.  Project-wide authorizations describe roles that users have been granted within a project. Basic default roles are "admin" and "basic-user".   Users with the role admin have greater capabilities to manipulate resources than users with the role basic-user.    New policies can be created, and different types of roles can be created to grant users specific capabilities.  For example, it would be possible (but not all that useful) to create a role that  allows creation of pods and prevents those pods from being deleted.

This functionality has been migrated into Kubernetes from OpenShift.

Build and deploy
^^^^^^^^^^^^^^^^

OpenShift has mechanisms to allow for automated build and deploy when certain events are triggered.  For example, if new pushes are made to the main branch of a repository, a build can be triggered, a new container will be created and then deployed to the cluster.

Monitoring
^^^^^^^^^^

The following monitoring tools were mentioned in the information for OpenShift, but not investigated.  Included is a brief synopsis of what was said.

Several different performance monitoring tools are recommended for use with OpenShift: 1) Grafana, Alertmanager, Prometheus (GAP); 2) Sysdig, 3) Datadog.

In the GAP solution, Prometheus is used to gather data which is converts into its own time-series database.  This can be set into Grafana via a Prometheus dashboard, and then attached to Alertmanager.  These solutions are open source.

Sysdig can be useful to debug issues that are being seen.  It can be invoked on the command line and can report statistics like process disk bandwidth use, which files are being utilized by a particular process most frequently, and  how much bandwidth processes are consuming.  Sysdig by itself is not a complete monitoring solution.   Sysdig Monitor exists, but is a commercial product.  Sysdig itself is open source.

DataDog provides application and service monitoring, along with alert monitoring and a UI.  It can also incorporate data from Nagios. DataDog is a commercial product and not open source.

LDF Criteria
^^^^^^^^^^^^

We have four main criteria for the LDF production cluster:

1) Control the proportion of resources dedicated to development, integration and production
2) Control the proportion of resources dedicated to each LSST component (eg, JupyterHub, Alert Distribution, etc)
3) Control over images that may be run in production in order to support a change policy which is to be determined.
4) Control the need of resources to be a) Unix root and 2) the Kubernetes equivalent of root controlling 1, 2 & 3 above.

OpenShift Projects and Kubernetes Namespaces each can be used to proportion resources in cases 1 and 2.

Once a change policy is in place, we will have procedures to determine which images are allowed to be put into the local container registry.   We can also set criteria on nodes to allow or disallow certain images from running on them, which addresses case 3.


We do not anticipate that any user another than the LSST system administration staff would need access to the system as root. The PodSecurityPolicy will be used to prevent containers running in a privileged context as root. Role-Based Access Control  (RBAC)will be used to enforce what types of resources can be accessed and utilized. RBAC controls allow us to create policies that give flexibility to users to act as OpenShift/Kubernetes admins without giving system administration (root) privileges to the underlying system.  This addresses case 4.



Conclusions
^^^^^^^^^^^

Our two main interests in looking at OpenShift are support for the LDF goals, and getting a software product that was going to be stable and easy to maintain.

Features that we're interested in using in LDF which originally existed in OpenShift have migrated into Kubernetes itself within the last few releases.   There is an advantage in that OpenShift is configured with RBAC at the time of installation, but this is can be replicated in the Kubernetes environment.

We plan on carefully vetting any containers that are deployed onto the cluster, and expect to have a smaller version of the cluster to test these before they are deployed.   These containers will likely be hosted with the Kubernetes cluster in a local, non-public private registry.  This will reduce the time delay in deployment, and eliminate the possibility of service outages  in an outside service which would cause downtime beyond our control.  We expect that containers will be publicly hosted on those outside services for third parties to use;  however, we 
don't want that to be the main dependency for our own cluster.   Additionally, we will still be able to configure the system to use outside container storage as a fallback position in case of an internal service outage.  

We are unlikely to utilize OpenShift build and deploy features.  We have our own internal build mechanism to verify that the software stack 
we're using is built properly and runs all internal tests. We will not deploy containers directly to the main Kubernetes cluster via a mechanism like this without vetting them on a separate cluster first, as mentioned earlier.

In the release briefing for Kubernetes 1.8, tighter integration with Prometheus was discussed, and this is still in development.   Along with the other LSST planned monitoring mechanisms (Nagios, etc), Prometheus seems adequate, at least initially.

It will be worthwhile to keep track of OpenShift and any advancements in it that may benefit the project.  It may also be better to use the OpenShift Enterprise version, for the reasons listed previously.  This would require support contracts and an outside dependency on RedHat, however.

As it stands now, we recommend that deploying Kubernetes itself would be better for the initial cluster installation.   

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :encoding: latex+latin
..    :style: lsst_aa
