FROM openfaas/of-watchdog:0.7.2 as watchdog
FROM golang:1.13-alpine3.11 as build

RUN apk --no-cache add git

ENV CGO_ENABLED=0

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

ENV CGO_ENABLED=0

RUN mkdir -p /go/src/handler
WORKDIR /go/src/handler
COPY . .

# Run a gofmt and exclude all vendored code.
#RUN test -z "$(gofmt -l $(find . -type f -name '*.go' -not -path "./vendor/*" -not -path "./function/vendor/*"))" || { echo "Run \"gofmt -s -w\" on your Golang code"; exit 1; }

ARG GO111MODULE="off"
ARG GOPROXY=""

RUN go build --ldflags "-s -w" -a -installsuffix cgo -o handler .
RUN go test handler/function/... -cover

FROM alpine:3.11
# Add non root user and certs
RUN apk --no-cache add ca-certificates \
    && addgroup -S app && adduser -S -g app app \
    && mkdir -p /home/app \
    && chown app /home/app



WORKDIR /home/app

COPY --from=builder /usr/bin/fwatchdog         .
COPY --from=builder /go/src/handler/function/  .
COPY --from=builder /go/src/handler/handler    .

RUN chown -R app /home/app

USER app

ENV fprocess="./handler"
EXPOSE 8080

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

CMD ["./fwatchdog"]
