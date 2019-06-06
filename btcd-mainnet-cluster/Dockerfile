FROM golang:1.11-alpine

# Install build dependencies such as git and glide.
RUN apk add --no-cache git gcc musl-dev

WORKDIR $GOPATH/src/github.com/btcsuite/btcd

# Grab and install the latest version of of btcd and all related dependencies.
RUN git clone https://github.com/btcsuite/btcd.git \
    && cd btcd \
    && git remote add conner https://github.com/cfromknecht/btcd.git \
    && git fetch conner btcd-reborn \
    && git checkout btcd-reborn \
    && GO111MODULE=on go mod vendor \
    && GO111MODULE=on go install -v -mod=vendor . ./cmd/...

# # Expose mainnet ports (server, rpc)
EXPOSE 8333 8334

COPY "start-btcctl.sh" .
COPY "start-btcd.sh" .

RUN apk add --no-cache \
    bash \
    ca-certificates \
&&  mkdir "/rpc" "/root/.btcd" "/root/.btcctl" \
&&  touch "/root/.btcd/btcd.conf" \
&&  chmod +x start-btcctl.sh \
&&  chmod +x start-btcd.sh \
# Manually generate certificate and add all domains, it is needed to connect
# "btcctl" and "lnd" to "btcd" over docker links.
&& gencerts --host="*" --directory="/rpc" --force

# Create a volume to house pregenerated RPC credentials. This will be
# shared with any lnd, btcctl containers so they can securely query btcd's RPC
# server.
# You should NOT do this before certificate generation!
# Otherwise manually generated certificate will be overridden with shared
# mounted volume! For more info read dockerfile "VOLUME" documentation.
VOLUME ["/rpc"]

CMD ["./start-btcd.sh"]