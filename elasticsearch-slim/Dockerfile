# 이미지 경량화를 위해 alpine 이미지 사용
ARG ALPINE_VERSION="3.21.2"

FROM alpine:$ALPINE_VERSION AS download-base
WORKDIR /downloads
RUN echo "$(apk --print-arch)" > /arch.txt 

FROM download-base AS installer
ARG ES_VERSION="7.10.2"

# elasticsearch 오픈 소스 최신 확인
# https://www.elastic.co/downloads/past-releases#elasticsearch-oss
# https://www.elastic.co/pricing/faq/licensing

RUN wget -O elasticsearch.tar.gz "https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-oss-${ES_VERSION}-linux-$(cat /arch.txt).tar.gz" 

RUN mkdir /elasticsearch && \
    tar -xzf elasticsearch.tar.gz --strip-components=1 -C /elasticsearch && \
    rm -rf /elasticsearch/jdk

# elasticsearch는 JVM을 필요로 하므로 경량 JRE 이미지 사용
# https://www.elastic.co/support/matrix#matrix_jvm
FROM openjdk:11.0.11-jre-slim

WORKDIR /usr/share/elasticsearch
COPY --from=installer /elasticsearch .

EXPOSE 9200 9300
ENV ES_HOME="/usr/share/elasticsearch" \
    ES_JAVA_OPTS="-Xms256m -Xmx256m"

COPY elasticsearch.yml log4j2.properties ./config/

RUN groupadd -g 1000 elasticsearch && \
    adduser -uid 1000 -gid 1000 --home ${ES_HOME} elasticsearch && \
    chmod 0775 ${ES_HOME} && \
    chown -R 1000:0 ${ES_HOME}

USER elasticsearch:root
CMD ["/usr/share/elasticsearch/bin/elasticsearch"]
