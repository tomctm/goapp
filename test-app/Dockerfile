FROM golang:1.20.3 as build-env

ENV APP_NAME golang-app
ENV CMD_PATH cmd/ops-test-app/main.go



COPY . $GOPATH/src/$APP_NAME
WORKDIR $GOPATH/src/$APP_NAME

RUN CGO_ENABLED=0 go build -v -o /$APP_NAME $GOPATH/src/$APP_NAME/$CMD_PATH


# Run Stage
FROM alpine:3.14
 
# Set environment variable
ENV APP_NAME golang-app
 
# Copy only required data into this image
COPY --from=build-env /$APP_NAME .
 
# Expose application port
EXPOSE 8080
 
# Start app
CMD ./$APP_NAME
