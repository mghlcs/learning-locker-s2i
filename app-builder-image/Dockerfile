FROM nodejs:10

ENV BUILDER_VERSION 0.1.0

LABEL maintainer="Michael Steigman <msteigman@mgh.harvard.edu>"
LABEL io.openshift.s2i.scripts-url="image:///opt/app-root/s2i/bin"
LABEL io.s2i.scripts-url="image:///opt/app-root/s2i/bin"

USER root

# The reason we're here.... Need these tools
RUN yum update \
    && yum install -y curl wget unzip procps-ng gettext \
    && yum --setopt=group_package_types=mandatory,default,optional groupinstall -y "Development Tools" \
    && yum clean all -y

# Rather than overwrite the existing scripts, change the location
COPY ./s2i/bin/ /opt/app-root/s2i/

USER 1001

EXPOSE 8080

CMD ["/usr/libexec/s2i/usage"]
