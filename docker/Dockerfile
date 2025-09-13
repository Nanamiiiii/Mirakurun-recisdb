ARG ARCH=

FROM ${ARCH}node:22.14.0-bookworm AS build
WORKDIR /app
ENV DOCKER=YES NODE_ENV=production
ADD . .
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends build-essential && \
    npm ci --include=dev && \
    npm run build && \
    npm ci --omit=dev

# Build & install recisdb
WORKDIR /src
RUN apt-get install -y --no-install-recommends git curl libclang-dev cmake libdvbv5-dev libpcsclite-dev libudev-dev pkg-config && \
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- --default-toolchain stable -y
RUN git clone --recursive https://github.com/kazuki0824/recisdb-rs.git && \
    cd recisdb-rs && \
    /root/.cargo/bin/cargo build -F dvb --release

# Build isdb-scanner
FROM python:3.11-bookworm AS py-builder
WORKDIR /src
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends build-essential git && \
    pip install poetry
RUN git clone https://github.com/tsukumijima/ISDBScanner.git -b v1.3.2 && \
    cd ISDBScanner && \
    poetry install && \
    poetry run pyinstaller --onefile --name=isdb-scanner isdb_scanner/__main__.py

# Runner

FROM ${ARCH}node:22.14.0-bookworm-slim
WORKDIR /app
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y --no-install-recommends \
        ca-certificates \
        make \
        gcc \
        g++ \
        pkg-config \
        pcscd \
        libpcsclite1 \
        libpcsclite-dev \
        libccid \
        libdvbv5-dev \
        pcsc-tools \
        dvb-tools \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

COPY --from=build /app /app
COPY --from=build /src/recisdb-rs/target/release/recisdb /usr/local/bin/
COPY --from=py-builder /src/ISDBScanner/dist/isdb-scanner /usr/local/bin/
CMD ["./docker/container-init.sh"]
EXPOSE 40772 9229
