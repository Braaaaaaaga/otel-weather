FROM golang:1.24-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -o /service-b .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
COPY --from=builder /service-b /service-b
EXPOSE 8081
ENTRYPOINT ["/service-b"]
