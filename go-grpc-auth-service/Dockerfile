############################
# STEP 1 build executable binary
############################
FROM golang:1.20-alpine3.18 as builder

RUN apk --no-cache update && apk --no-cache add git tzdata 

# Create appuser.
ENV USER=appuser
ENV UID=10001

RUN adduser \    
  --disabled-password \    
  --gecos "" \    
  --home "/nonexistent" \    
  --shell "/sbin/nologin" \    
  --no-create-home \    
  --uid "${UID}" \    
  "${USER}"

# Setting timezone
ENV TZ=Asia/Jakarta
RUN ln -s /usr/share/zoneinfo/$TZ /etc/localtime

# Set default working directory of container
WORKDIR $GOPATH/src/github.com/azkanurhuda/go-grpc-product-microservices

# Copy all
COPY . .


# Copy env file
RUN mkdir -p /go/bin/pkg/config/envs
RUN cp ./pkg/config/envs/* /go/bin/pkg/config/envs

# Build an excutable app
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
  -ldflags="-w -s" -o /go/bin/auth ./cmd

############################
# STEP 2 build a small image
############################
FROM scratch AS base

# Add Maintainer info
LABEL maintainer="Azka <nurhudaazka@gmail.com>"

# Setting timezone
ENV TZ=Asia/Jakarta
COPY --from=builder /usr/share/zoneinfo /usr/share/zoneinfo
COPY --from=builder /etc/localtime /etc/localtime

# Import the user and group files from the builder.
COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /etc/group /etc/group

# Copy excutable app and env from builder stage to base stage
COPY --from=builder /go/bin/auth /go/bin/auth
COPY --from=builder /go/bin/pkg/config/envs/dev.env /pkg/config/envs/dev.env

# Set default user
USER appuser:appuser

# Expose app port
EXPOSE 50051

# Run app
ENTRYPOINT ["/go/bin/auth"]