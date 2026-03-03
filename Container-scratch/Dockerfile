FROM alpine:3.20 AS builder
RUN apk add --no-cache curl
RUN curl -fsSL https://github.com/NetApp/tridentctl-protect/releases/download/25.10.0/tridentctl-protect-linux-amd64 \
    -o /tridentctl-protect && chmod +x /tridentctl-protect

FROM scratch
COPY --from=builder /tridentctl-protect /tridentctl-protect
ENTRYPOINT ["/tridentctl-protect"]