FROM ubuntu:25.10

RUN apt-get update \
    && apt-get install -y --no-install-recommends libatomic1 ca-certificates \
    && rm -rf /var/lib/apt/lists/*

ADD https://github.com/Kong/ngx_wasm_module/releases/download/prerelease-0.6.0/wasmx-prerelease-0.6.0-v8-x86_64-ubuntu22.04.tar.gz /tmp/wasmx.tar.gz

RUN mkdir -p /etc/nginx \
    && tar -xvf /tmp/wasmx.tar.gz -C /tmp \
    && mv /tmp/wasmx-prerelease-0.6.0-v8-x86_64-ubuntu22.04/* /etc/nginx/ \
    && rm -rf /tmp/wasmx.tar.gz /tmp/wasmx-prerelease-0.6.0-v8-x86_64-ubuntu22.04

RUN mkdir -p /wasm

COPY wasm/sablierproxywasm.wasm /wasm/sablierproxywasm.wasm

WORKDIR /etc/nginx

CMD ["./nginx", "-g", "daemon off;"]