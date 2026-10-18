# Sandbox image exercised by the nightly zero-warning build via ci-docker.yml.
# Builds an image, then runs a test command inside the container.
FROM alpine:3.20

# hadolint ignore=DL3015
RUN apk add --no-cache bash coreutils

COPY test.sh /test.sh
RUN chmod +x /test.sh

CMD ["sh", "/test.sh"]
