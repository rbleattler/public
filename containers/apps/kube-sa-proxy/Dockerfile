# Stage 1 - Build the Go application
FROM golang:1.24.4-alpine@sha256:68932fa6d4d4059845c8f40ad7e654e626f3ebd3706eef7846f319293ab5cb7a AS builder

# Install necessary build dependencies
RUN apk --no-cache add --update gcc musl-dev

# Create the necessary directories
RUN mkdir -p /build /output

# Set the working directory
WORKDIR /build

# Copy go mod and sum files
COPY ./containers/apps/kube-sa-proxy/go.mod ./containers/apps/kube-sa-proxy/go.sum ./

# Download dependencies
RUN go mod download

# Copy the rest of the Go application source code
COPY ./containers/apps/kube-sa-proxy/cmd/main.go .
COPY ./containers/apps/kube-sa-proxy/internal/config ./internal/config
COPY ./containers/apps/kube-sa-proxy/internal/proxy ./internal/proxy
COPY ./containers/apps/kube-sa-proxy/internal/utils ./internal/utils

# Build the Go application
RUN go build -ldflags "-w -s" -o /output/my-proxy-service .

# Stage 2 - Create the final image
FROM alpine@sha256:8a1f59ffb675680d47db6337b49d22281a139e9d709335b492be023728e11715 AS runner

# Install necessary runtime dependencies
RUN apk --no-cache add ca-certificates

# Set the working directory
WORKDIR /app

# Copy the binary from the builder stage
COPY --from=builder /output/my-proxy-service .

# Set environment variables
ENV PORT=3000

# Expose the port
EXPOSE $PORT

# Set the default command to run the binary
CMD sh -c "./my-proxy-service"
