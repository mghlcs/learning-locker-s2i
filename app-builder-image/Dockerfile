FROM nodejs:10

ENV BUILDER_VERSION 0.1.0

LABEL maintainer="Michael Steigman <msteigman@mgh.harvard.edu>"
LABEL io.openshift.s2i.scripts-url="image:///opt/app-root/s2i/bin"
LABEL io.s2i.scripts-url="image:///opt/app-root/s2i/bin"

USER root

RUN yum -y --setopt=tsflags=nodocs --enablerepo="rhel-server-rhscl-7-rpms" install devtoolset-7-toolchain \
    && yum -y clean all

USER 1001

COPY ./s2i/bin/ /opt/app-root/s2i/

RUN source scl_source enable devtoolset-7

EXPOSE 8080

CMD ["/opt/app-root/s2i/bin/usage"]
