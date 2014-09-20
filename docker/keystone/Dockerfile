FROM fedora-rdo-base
MAINTAINER jlabocki@redhat.com

RUN yum install -y openstack-keystone && yum clean all

ADD ./start.sh /start.sh

CMD ["/start.sh"]
