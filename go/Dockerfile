ARG GO_VERSION=1.22
ARG ALPINE_VERSION=3.20

FROM --platform=${BUILDPLATFORM:-linux/amd64} golang:${GO_VERSION}-alpine${ALPINE_VERSION} AS build

ARG TARGETOS 
ARG TARGETARCH

# If CGO is required
# RUN apk add --no-cache build-base

# Switch into workspace
WORKDIR /workspace

# Copy Go module files into the container's workspace
COPY go.mod go.sum ./

# Install dependencies
RUN go mod download

# Copy the local package files to the container's workspace
COPY . .

# Build the application into a static executable while removing the symbol table and debugging information
RUN CGO_ENABLED=0 GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build -ldflags="-s -w" -o build/server .
# RUN CGO_ENABLED=1 GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build -o build/server -ldflags "-s -w -linkmode 'external' -extldflags '-static'" .

FROM gcr.io/distroless/static:nonroot

# Switch into workspace
WORKDIR /app

# Copy Executable from build
COPY --from=build /workspace/build/server /app/

# Start the server application
CMD ["/app/server"]
