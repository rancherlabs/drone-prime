#FROM alpine:3.10 as base

#ENV MANIFEST_TOOL_VERSION 1.0.2

#RUN apk add --no-cache curl && \
  #curl -sSLo /bin/manifest-tool https://github.com/estesp/manifest-tool/releases/download/v${MANIFEST_TOOL_VERSION}/manifest-tool-linux-amd64 && \
  #chmod +x /bin/manifest-tool

#FROM plugins/base:multiarch

##LABEL maintainer="Drone.IO Community <drone-dev@googlegroups.com>" \
  ##org.label-schema.name="Drone Manifest" \
  ##org.label-schema.vendor="Drone.IO Community" \
  ##org.label-schema.schema-version="1.0"

#COPY --from=base /bin/manifest-tool /bin/

#ADD release/linux/amd64/drone-manifest /bin/
#ENTRYPOINT ["/bin/drone-manifest"]


ARG GO_IMAGE=registry.suse.com/bci/golang:1.17
ARG DIND_IMAGE=docker:20.10.17-dind

FROM ${GO_IMAGE} AS build
ARG upstream=https://github.com/drone-plugins/drone-manifest.git
ARG version=v1.2.3

ENV MANIFEST_TOOL_VERSION 1.0.2

WORKDIR /src
run git clone --depth=1 --branch=${version} ${upstream} .
run GOOS=linux GOARCH=amd64 CGO_ENABLED=0 GO111MODULE=on go build -v -a -tags netgo -o release/linux/amd64/drone-manifest
RUN go install github.com/awslabs/amazon-ecr-credential-helper/ecr-login/cli/docker-credential-ecr-login@latest
RUN curl -sSLo /bin/manifest-tool https://github.com/estesp/manifest-tool/releases/download/v${MANIFEST_TOOL_VERSION}/manifest-tool-linux-amd64 && \
  chmod +x /bin/manifest-tool

FROM ${DIND_IMAGE}

WORKDIR /src/
#COPY --from=build /src/release/drone-docker /bin/
COPY --from=build /bin/manifest-tool /bin/
COPY --from=build /go/bin/docker-credential-ecr-login /bin/
COPY --from=build /src/release/linux/amd64/drone-manifest /bin/
RUN mkdir /root/.docker
COPY ../config.json /root/.docker/config.json

ENV DOCKER_HOST=unix:///var/run/docker.sock
ENV METHOD=get \
    AWS_ECR_DISABLE_CACHE=True
#ENTRYPOINT ["/bin/sh", "dockerd-entrypoint.sh", "/bin/drone-docker"]
ENTRYPOINT ["/bin/drone-manifest"]