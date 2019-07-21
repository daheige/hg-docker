FROM golang:1.12  AS go-builder
WORKDIR /my-go
#RUN mkdir -p /my-go  #会自动创建/my-go

COPY app.go /my-go/
#禁用cgo模块，开始go mod auto机制
RUN CGO_ENABLED=0 GOOS=linux GO111MODULE=auto \
    GOPROXY=https://mirrors.aliyun.com/goproxy/ && \
    cd /my-go && go mod init my-go && \
    go mod tidy && go build -o app .


FROM alpine:3.10
WORKDIR /root #作为COPY CMD工作目录

RUN apk --no-cache add ca-certificates

#如果需要bash,curl请用下面的RUN
#RUN echo "https://mirror.tuna.tsinghua.edu.cn/alpine/v3.10/main/" > /etc/apk/repositories \
#        && apk update \
#        && apk upgrade \
#        && apk add --no-cache ca-certificates bash \
#        bash-doc \
#        bash-completion curl \
#        && rm -rf /var/cache/apk/*

# copy file from go-builder
COPY --from=go-builder /my-go/app /root
CMD ["/root/app"]
