# 이미지 경량화를 위한 alpine 선택
ARG NODE_TAG=10.23.1-alpine3.11
ARG ALPINE_VERSION="3.21.2"

FROM alpine:$ALPINE_VERSION AS download-base
WORKDIR /downloads
RUN echo "$(apk --print-arch)" > /arch.txt 

FROM download-base AS installer
ARG KIBANA_VERSION="7.10.2" 

# Kibana 오픈 소스 가장 최근 버전
# https://www.elastic.co/downloads/past-releases#kibana-oss
# https://www.elastic.co/pricing/faq/licensing

RUN wget -O kibana.tar.gz https://artifacts.elastic.co/downloads/kibana/kibana-oss-${KIBANA_VERSION}-linux-$(cat /arch.txt).tar.gz

RUN mkdir /kibana && \
    tar -xzf kibana.tar.gz --strip-components=1 -C /kibana && \
    rm -rf /kibana/node

# Kibana는 node.js를 필요로 하므로 공식 node.js 이미지 사용
# https://www.elastic.co/guide/en/kibana/master/upgrading-nodejs.html
FROM node:$NODE_TAG

EXPOSE 5601
ENV KIBANA_HOME="/usr/share/kibana" 

WORKDIR /usr/share/kibana
COPY --from=installer /kibana .
COPY ./kibana bin/
COPY ./kibana.yml config/

RUN chmod +x bin/kibana
CMD ["/usr/share/kibana/bin/kibana", "--allow-root"]
