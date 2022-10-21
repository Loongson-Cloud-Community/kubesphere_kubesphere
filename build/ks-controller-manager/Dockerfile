# Copyright 2020 The KubeSphere Authors. All rights reserved.
# Use of this source code is governed by an Apache license
# that can be found in the LICENSE file.

# Download dependencies
FROM cr.loongnix.cn/library/alpine:3.11 as base_os_context

ARG TARGETARCH
ARG TARGETOS
ARG HELM_VERSION=v3.5.2
ARG KUSTOMIZE_VERSION=v4.2.0
ARG INGRESS_NGINX_VERSION=3.35.0

ENV OUTDIR=/out
RUN mkdir -p ${OUTDIR}/usr/local/bin
RUN mkdir -p ${OUTDIR}/var/helm-charts

WORKDIR /tmp

RUN apk add --no-cache ca-certificates wget

# Install helm
#ADD https://get.helm.sh/helm-${HELM_VERSION}-${TARGETOS}-${TARGETARCH}.tar.gz /tmp
#RUN tar xvzf /tmp/helm-${HELM_VERSION}-${TARGETOS}-${TARGETARCH}.tar.gz -C /tmp
#RUN mv /tmp/${TARGETOS}-${TARGETARCH}/helm ${OUTDIR}/usr/local/bin/
RUN wget -q http://10.130.0.16:8000/dist/helm/v3.4.5/helm -O ${OUTDIR}/usr/local/bin/helm && chmod +x ${OUTDIR}/usr/local/bin/helm

# install kustomize
#ADD https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2F${KUSTOMIZE_VERSION}/kustomize_${KUSTOMIZE_VERSION}_${TARGETOS}_${TARGETARCH}.tar.gz /tmp
#RUN tar xvzf /tmp/kustomize_${KUSTOMIZE_VERSION}_linux_${TARGETARCH}.tar.gz -C /tmp
#RUN mv /tmp/kustomize ${OUTDIR}/usr/local/bin/
RUN wget -q http://10.130.0.16:8000/dist/kubernetes-sigs/kustomize/kustomize-loong64-v4.2.0 -O ${OUTDIR}/usr/local/bin/kustomize && chmod +x ${OUTDIR}/usr/local/bin/kustomize

# Install Nginx Ingress Helm Chart
ADD http://10.130.0.16:8000/dist/kubernetes/ingress-nginx/ingress-nginx-3.35.0.tgz /tmp
RUN tar xvzf /tmp/ingress-nginx-${INGRESS_NGINX_VERSION}.tgz -C /tmp
RUN mv /tmp/ingress-nginx ${OUTDIR}/var/helm-charts/

# Build

FROM cr.loongnix.cn/library/golang:1.19-alpine as build_context

ENV OUTDIR=/out
RUN mkdir -p ${OUTDIR}/usr/local/bin/
RUN mkdir -p ${OUTDIR}/var/helm-charts

WORKDIR /workspace
ADD . /workspace/

#RUN make ks-controller-manager
#RUN mv /workspace/bin/cmd/controller-manager ${OUTDIR}/usr/local/bin/
RUN wget -q http://10.130.0.16:8000/dist/kubesphere/kubesphere/controller-manager-loong64-v3.2.1 -O ${OUTDIR}/usr/local/bin/ks-controller-manager && chmod +x ${OUTDIR}/usr/local/bin/ks-controller-manager

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
