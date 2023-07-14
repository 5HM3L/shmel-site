FROM amd64/ubuntu as STAGEONE

ENV HUGO_VERSION=0.104.3
ADD https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_${HUGO_VERSION}_linux-arm64.tar.gz /tmp/
RUN tar -xf /tmp/hugo_${HUGO_VERSION}_linux-arm64.tar.gz -C /usr/local/bin/

RUN apt-get update
RUN apt-get install -y python3-pygments

COPY src /src
RUN hugo --source=/src/ --destination=/public/

FROM amd64/nginx:stable-alpine3.17
RUN apk --update add curl bash
RUN rm /etc/nginx/conf.d/default.conf
COPY docker/nginx/nginx.conf /etc/nginx/nginx.conf
COPY --from=STAGEONE /public/ /app
EXPOSE 80