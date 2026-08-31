# syntax=docker/dockerfile:1
ARG UPSTREAM_REF=ad9ce203cbb7fdb54f9404a7abb2df605893f388

FROM alpine/git:2.47.2 AS source
ARG UPSTREAM_REF
RUN git clone https://github.com/qianlipp/fn-music-bridge.git /src \
    && cd /src \
    && git checkout "${UPSTREAM_REF}" \
    && rm -rf .git docs package-src dist scripts

FROM golang:1.23-alpine AS api-builder
COPY --from=source /src/api /src/api
WORKDIR /src/api
RUN CGO_ENABLED=0 GOOS=linux go test ./... \
    && CGO_ENABLED=0 GOOS=linux go build -trimpath -ldflags="-s -w" -o /out/yinqiao-api ./cmd/yinqiao-api

FROM golang:1.23-alpine AS service-builder
COPY --from=source /src/service /src/service
WORKDIR /src/service
RUN CGO_ENABLED=0 GOOS=linux go test ./... \
    && CGO_ENABLED=0 GOOS=linux go build -trimpath -ldflags="-s -w" -o /out/yinqiao-service ./cmd/yinqiao-service

FROM alpine:3.21 AS api
RUN apk add --no-cache ca-certificates tzdata \
    && addgroup -S yinqiao && adduser -S -G yinqiao yinqiao
COPY --from=api-builder /out/yinqiao-api /usr/local/bin/yinqiao-api
USER yinqiao
EXPOSE 18090
ENTRYPOINT ["/usr/local/bin/yinqiao-api"]

FROM alpine:3.21 AS service
RUN apk add --no-cache ca-certificates tzdata \
    && addgroup -S yinqiao && adduser -S -G yinqiao yinqiao \
    && mkdir -p /data && chown yinqiao:yinqiao /data
COPY --from=service-builder /out/yinqiao-service /usr/local/bin/yinqiao-service
USER yinqiao
VOLUME ["/data"]
EXPOSE 14040
ENTRYPOINT ["/usr/local/bin/yinqiao-service"]
