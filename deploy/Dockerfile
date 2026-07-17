# Build go
FROM golang:1.25.0-alpine AS builder
WORKDIR /app
COPY . .
ENV CGO_ENABLED=0
RUN GOEXPERIMENT=jsonv2 GOPROXY=https://goproxy.cn,direct go mod download
RUN GOEXPERIMENT=jsonv2 go build -v -o V2bX -tags "sing xray hysteria2 with_quic with_grpc with_utls with_wireguard with_acme with_gvisor"

# Release
FROM  alpine
WORKDIR /etc/V2bX
# Copy only requirements and deploy files first to leverage Docker cache
COPY deploy/entrypoint.sh deploy/config.json deploy/nginx/ example/* ./

# 安装必要的工具包
# Configure timezone, install runtime deps and python packages in a single layer
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
&& echo "Asia/Shanghai" > /etc/timezone \
&& sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories \
&& apk --update --no-cache add tzdata ca-certificates nginx nginx-mod-stream\
#  && apk add --no-cache bash vim curl net-tools \
&& rm -rf /var/cache/apk/* /etc/nginx/http.d/* /etc/nginx/conf.d/* \
&& chmod +x /etc/V2bX/entrypoint.sh \
&& mv /etc/V2bX/html /etc/nginx


COPY --from=builder /app/V2bX /etc/V2bX

ENV LANG=zh_CN.UTF-8
EXPOSE 443
ENTRYPOINT ["sh", "/etc/V2bX/entrypoint.sh"]