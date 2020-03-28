FROM nodejs:10

ENV BUILDER_VERSION 0.1.0

LABEL maintainer="Michael Steigman <msteigman@mgh.harvard.edu>"
LABEL io.openshift.s2i.scripts-url="image:///opt/app-root/s2i/bin"
LABEL io.s2i.scripts-url="image:///opt/app-root/s2i/bin"

USER root

RUN yum -y --disablerepo=rhel-7-server-rpms --enablerepo=ubi-7 update \
    && yum -y --disablerepo=rhel-7-server-rpms --enablerepo=ubi-7 --enablerepo=ubi-7-server-devtools-rpms \
       group install "Development tools" --setopt=group_package_types=mandatory,default,optional
    && yum -y clean all

# Change script location rather than overwrite originals
COPY ./s2i/bin/ /opt/app-root/s2i/

USER 1001

EXPOSE 8080

CMD ["/usr/libexec/s2i/usage"]
