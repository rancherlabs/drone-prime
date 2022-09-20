ARG GO_IMAGE=registry.suse.com/bci/golang:1.17
ARG DIND_IMAGE=docker:20.10.17-dind

FROM ${GO_IMAGE} AS build
#ARG UPSTREAM=HTTPS://GITHUB.COM/DRONE-PLUGINS/DRONE-MANIFEST.GIT
#ARG VERSION=V1.2.3
ARG upstream=https://github.com/Anarkis/drone-manifest.git
ARG version=v0.0.10
ARG arch=linux-amd64

ENV MANIFEST_TOOL_VERSION 2.0.5

WORKDIR /src
RUN zypper install -y tar wget gzip
RUN git clone --depth=1 --branch=${version} ${upstream} .
RUN GOOS=linux GOARCH=amd64 CGO_ENABLED=0 GO111MODULE=on go build -v -a -tags netgo -o release/linux/amd64/drone-manifest

RUN go install github.com/awslabs/amazon-ecr-credential-helper/ecr-login/cli/docker-credential-ecr-login@latest
RUN wget https://github.com/estesp/manifest-tool/releases/download/v${MANIFEST_TOOL_VERSION}/binaries-manifest-tool-${MANIFEST_TOOL_VERSION}.tar.gz && \
  tar -zxvf binaries-manifest-tool-${MANIFEST_TOOL_VERSION}.tar.gz manifest-tool-${arch}

FROM ${DIND_IMAGE}

WORKDIR /src/
COPY --from=build /src/manifest-tool-linux-amd64 /bin/manifest-tool
COPY --from=build /go/bin/docker-credential-ecr-login /bin/
COPY --from=build /src/release/linux/amd64/drone-manifest /bin/
RUN mkdir /root/.docker
COPY config.json /root/.docker/config.json

ENV DOCKER_HOST=unix:///var/run/docker.sock
ENV METHOD=get \
    AWS_ECR_DISABLE_CACHE=True
ENV PUBLIC_ECR=true
ENTRYPOINT ["/bin/drone-manifest"]
