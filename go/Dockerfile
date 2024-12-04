# syntax=docker/dockerfile:1

ARG GO_VERSION=1.23
ARG ALPINE_VERSION=3.20

FROM --platform=${BUILDPLATFORM:-linux/amd64} golang:${GO_VERSION}-alpine${ALPINE_VERSION} AS build

ARG TARGETOS
ARG TARGETARCH

# If CGO is required
# RUN apk add --no-cache build-base

# Switch into workspace
WORKDIR /workspace

# Install dependencies
RUN --mount=type=cache,target=/go/pkg/mod \
    --mount=type=bind,source=go.mod,target=go.mod \
    --mount=type=bind,source=go.sum,target=go.sum \
    go mod download -x

# Build the application into a static executable while removing the symbol table and debugging information
RUN --mount=type=bind,target=. \
    --mount=type=cache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    CGO_ENABLED=0 GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build -ldflags="-s -w" -o /build/server .
# CGO_ENABLED=1 GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build -ldflags "-s -w -linkmode 'external' -extldflags '-static'" -o /build/server .

FROM gcr.io/distroless/static:nonroot

# Switch into workspace
WORKDIR /app

# Copy Executable from build
COPY --from=build /build/server /app/

# Start the server application
CMD ["/app/server"]
