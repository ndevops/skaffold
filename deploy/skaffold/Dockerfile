# Copyright 2018 The Skaffold Authors All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM gcr.io/gcp-runtimes/ubuntu_16_0_4 as runtime_deps

RUN apt-get update && \
    apt-get install --no-install-recommends --no-install-suggests -y \
        python-dev

ENV KUBECTL_VERSION v1.10.0
RUN curl -Lo /usr/local/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl  && \
    chmod +x /usr/local/bin/kubectl

ENV HELM_VERSION v2.8.1
RUN curl -LO https://storage.googleapis.com/kubernetes-helm/helm-${HELM_VERSION}-linux-amd64.tar.gz && \
    tar -zxvf helm-${HELM_VERSION}-linux-amd64.tar.gz && \
    mv linux-amd64/helm /usr/local/bin/helm

ENV CLOUD_SDK_VERSION 217.0.0
RUN curl -LO https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-${CLOUD_SDK_VERSION}-linux-x86_64.tar.gz && \
 tar -zxvf google-cloud-sdk-${CLOUD_SDK_VERSION}-linux-x86_64.tar.gz && \
    CLOUDSDK_PYTHON="python2.7" /google-cloud-sdk/install.sh --usage-reporting=false \
        --bash-completion=false \
        --disable-installation-options
ENV PATH=$PATH:/google-cloud-sdk/bin
RUN /google-cloud-sdk/bin/gcloud auth configure-docker

ENV KUSTOMIZE_VERSION=1.0.1
RUN curl -LO https://github.com/kubernetes-sigs/kustomize/releases/download/v${KUSTOMIZE_VERSION}/kustomize_${KUSTOMIZE_VERSION}_linux_amd64 && \
    chmod +x kustomize_${KUSTOMIZE_VERSION}_linux_amd64 && \
    mv kustomize_${KUSTOMIZE_VERSION}_linux_amd64 /usr/local/bin/kustomize

RUN echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | tee /etc/apt/sources.list.d/bazel.list \
  && curl https://bazel.build/bazel-release.pub.gpg | apt-key add -

RUN apt-get update \
  && apt-get install -y bazel

ENV PATH /usr/local/go/bin:/go/bin:/google-cloud-sdk/bin:$PATH

FROM runtime_deps as builder

ENV ASCIIDOCTOR_VERSION=1.5.7.1
ENV ASCIIDOCTOR_PDF_VERSION=1.5.0.alpha.16
RUN apt-get install --no-install-recommends --no-install-suggests -y \
        ca-certificates \
        curl \
        build-essential \
        git \
        gcc \
        python-setuptools \
        lsb-release \
        software-properties-common \
        ruby-dev \
        apt-transport-https && \
    gem install  --no-document "asciidoctor-pdf:${ASCIIDOCTOR_PDF_VERSION}" \
        asciidoctor:${ASCIIDOCTOR_VERSION} \
        pygments.rb rouge && \
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - && \
    apt-key fingerprint 0EBFCD88 && \
    add-apt-repository \
           "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
           xenial \
           edge" && \
    apt-get -y update && \
    apt-get -y install docker-ce=17.12.0~ce-0~ubuntu

COPY --from=golang:1.10 /usr/local/go /usr/local/go
ENV PATH /usr/local/go/bin:/go/bin:$PATH
ENV GOPATH /go/

WORKDIR /go/src/github.com/GoogleContainerTools/skaffold

COPY . .

FROM builder as integration
ARG VERSION

RUN make out/skaffold-linux-amd64 VERSION=$VERSION && mv out/skaffold-linux-amd64 /usr/bin/skaffold

CMD ["make", "integration"]

FROM runtime_deps as distribution

COPY --from=integration /usr/bin/skaffold /usr/bin/skaffold

