FROM alpine:latest

# set labels for metadata
LABEL maintainer="Brandon Wilson<brandon@coil.com>" \
  name="name-mutator" \
  description="A Kubernetes mutating webhook server that changes resource name to hash of spec" \
  summary="A Kubernetes mutating webhook server that changes resource name to hash of spec"

# set environment variables
ENV NAME_MUTATOR=/usr/local/bin/name-mutator \
  USER_UID=1001 \
  USER_NAME=name-mutator

# install name-mutator binary
COPY build/_output/bin/name-mutator ${NAME_MUTATOR}

# copy licenses
RUN mkdir /licenses
COPY LICENSE /licenses

# set entrypoint
ENTRYPOINT ["/usr/local/bin/name-mutator"]

# switch to non-root user
USER ${USER_UID}
