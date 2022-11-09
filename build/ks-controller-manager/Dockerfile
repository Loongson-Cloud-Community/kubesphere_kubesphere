# Copyright 2020 The KubeSphere Authors. All rights reserved.
# Use of this source code is governed by an Apache license
# that can be found in the LICENSE file.

# Download dependencies
FROM cr.loongnix.cn/library/alpine:3.11 as base_os_context

ARG TARGETARCH
ARG TARGETOS
ARG HELM_VERSION=v3.5.2
ARG KUSTOMIZE_VERSION=v4.2.0
ARG INGRESS_NGINX_VERSION=4.0.13

ENV OUTDIR=/out
RUN mkdir -p ${OUTDIR}/usr/local/bin
RUN mkdir -p ${OUTDIR}/var/helm-charts

WORKDIR /tmp

RUN apk add --no-cache ca-certificates wget

RUN https_proxy=http://10.130.0.16:7890 wget -q https://github.com/Loongson-Cloud-Community/helm/releases/download/v3.9.0/helm-loong64-3.9.0-static -O ${OUTDIR}/usr/local/bin/helm && chmod +x ${OUTDIR}/usr/local/bin/helm

# 这里的链接需要修改
RUN wget -q http://10.130.0.16:8000/dist/kubernetes-sigs/kustomize/kustomize-loong64-v4.2.0 -O ${OUTDIR}/usr/local/bin/kustomize && chmod +x ${OUTDIR}/usr/local/bin/kustomize

# 这里的链接需要修改
ADD http://10.130.0.16:8000/dist/kubernetes%2Fingress-nginx/ingress-nginx-4.0.13.tgz /tmp
RUN tar xvzf /tmp/ingress-nginx-${INGRESS_NGINX_VERSION}.tgz -C /tmp
RUN mv /tmp/ingress-nginx ${OUTDIR}/var/helm-charts/

# Build

FROM cr.loongnix.cn/library/golang:1.19-alpine as build_context

ENV OUTDIR=/out
RUN mkdir -p ${OUTDIR}/usr/local/bin/
RUN mkdir -p ${OUTDIR}/var/helm-charts

WORKDIR /workspace
ADD . /workspace/
RUN apk add --no-cache make bash git

RUN make ks-controller-manager
RUN mv /workspace/bin/cmd/controller-manager ${OUTDIR}/usr/local/bin/

# Copy gateway config and helm chart
RUN mv /workspace/config/gateway ${OUTDIR}/var/helm-charts/
RUN mv /workspace/config/watches.yaml ${OUTDIR}/var/helm-charts/

# Final Image

FROM cr.loongnix.cn/library/alpine:3.11 

COPY --from=base_os_context /out/ /
COPY --from=build_context /out/ /

WORKDIR /

EXPOSE 8443 8080

CMD ["sh"]
