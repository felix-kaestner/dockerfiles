ARG GO_VERSION=1.18
ARG ALPINE_VERSION=3.16

FROM --platform=${BUILDPLATFORM:-linux/amd64} golang:${GO_VERSION}-alpine${ALPINE_VERSION} AS build

ARG TARGETOS 
ARG TARGETARCH

ENV CGO_ENABLED=0

# Switch into workspace
WORKDIR /workspace

# Copy Go module files into the container's workspace
COPY go.mod go.sum ./

# Install dependencies
RUN go mod download

# Copy the local package files to the container's workspace
COPY . .

# Build the application
RUN GOOS=${TARGETOS} GOARCH=${TARGETARCH} go build -o build/server .

FROM gcr.io/distroless/static:nonroot

# Switch into workspace
WORKDIR /app

# Copy Executable from build
COPY --from=build /workspace/build/server /app/

# Start the server application
CMD ["/app/server"]