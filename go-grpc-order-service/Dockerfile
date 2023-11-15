############################
# STEP 1 build executable binary
############################
FROM golang:alpine AS builder

RUN apk --no-cache update && apk --no-cache add build-base git coreutils

# Create appuser.
ENV USER=appuser
ENV UID=10001

# See https://stackoverflow.com/a/55757473/12429735RUN 
RUN adduser \    
  --disabled-password \    
  --gecos "" \    
  --home "/nonexistent" \    
  --shell "/sbin/nologin" \    
  --no-create-home \    
  --uid "${UID}" \    
  "${USER}"

WORKDIR $GOPATH/src/github.com/azkanurhuda/go-grpc-product-microservices

COPY . .

RUN go build -tags static_all -o /go/bin/order ./cmd

############################
# STEP 2 build a small image
############################
FROM scratch

# Add Maintainer info
LABEL maintainer="Azka <nurhudaazka@gmail.com>"

# Setting timezone

COPY --from=builder /usr/local/go/lib/time/zoneinfo.zip /
ENV TZ=Asia/Jakarta
ENV ZONEINFO=/zoneinfo.zip

# Import the user and group files from the builder.
COPY --from=builder /etc/passwd /etc/passwd
COPY --from=builder /etc/group /etc/group

COPY --from=builder /go/bin/order /go/bin/order

USER appuser

# Download necessary Go modules
EXPOSE $PORT

ENTRYPOINT ["/go/bin/order"]