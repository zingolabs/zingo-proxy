# Zingo-Proxy
A(n eventual) replacement for lightwalletd, written in Rust.

Currently connects to a lightwalletd, and acts as a man-in-the-middle proxy that does nothing. 
Each RPC we wish to support will be added individually, by connecting to zcashd/zebrad and doing any nessisary processing.
Eventually, we'll no longer have any calls that need to use the lightwalletd, and it can be removed from the network stack entirely.

A note to developers/consumers/contributers: The end goal is not an exact one-to-one port of all existing lwd functionaliy.
We seek to have at least the minimal functionality nessisary for zingo to connect to zingoproxy instead of a lightwalletd, 
and continue to implement any useful caching/preprocessing we can add...but full backwards compatibilty with all preexisting lightwalletd RPCs is not likely.

# Dependencies
1) zebrad <https://github.com/ZcashFoundation/zebra.git>
2) lightwalletd <https://github.com/zcash/lightwalletd.git>
3) zingolib <https://github.com/zingolabs/zingolib.git> [if running zingo-cli]

# zproxy
- To run tests:
1) Run `$ zebrad --config #PATH_TO_ZINGO_PROXY/zebrad.toml start`
2) Run `$ ./lightwalletd --no-tls-very-insecure --zcash-conf-path $PATH_TO_ZINGO_PROXY/zcash.conf --data-dir . --log-file /dev/stdout`
3) Run `$ cargo nextest run`

- To run zingo-cli through zingo-proxy:
1) Run `$ zebrad --config #PATH_TO_ZINGO_PROXY/zebrad.toml start`
2) Run `$ ./lightwalletd --no-tls-very-insecure --zcash-conf-path $PATH_TO_ZINGO_PROXY/zcash.conf --data-dir . --log-file /dev/stdout`
3) Run `$ cargo run --bin zproxy`
3) Run `$ cargo run --release --package zingo-cli -- --chain "testnet" --server "127.0.0.1:8080" --data-dir ~/wallets/test_wallet`

# Nym-Proxy / Nym-Server
A nym powered proxy between zcash wallets and lightwalletd, currently capable of sending transactions.

This is the POC and initial work on enabling zcash infrastructure to use the nym mixnet.

[Nym_POC](./docs/nym_poc.pdf) shows the current state of this work ands our vision for the future. 

Our plan is to first enable wallets to send and recieve transactions via a nym powered proxy between wallets and a lightwalletd before looking at the wider zcash ecosystem.

# nproxy/nserver
- To run zingo-cli through nym-proxy/server, connecting to lightwalletd/zebrad locally:
1) Run `$ zebrad --config #PATH_TO_ZINGO_PROXY/zebrad.toml start`
2) Run `$ ./lightwalletd --no-tls-very-insecure --zcash-conf-path $PATH_TO_ZINGO_PROXY/zcash.conf --data-dir . --log-file /dev/stdout`
3) Run `$ cargo run --bin nserver`
4) Copy nym address displayed
5) Run `$ cargo run --bin nproxy "nserver address copied"`
6) Run `$ cargo run --release --package zingo-cli -- --chain "testnet" --server "127.0.0.1:8080" --data-dir ~/wallets/testnet_wallet`

- Nym-proxy/server can also be set up to connect directly to the official lightwalletd server running mainnet:
1) Two values, in src/nproxy.rs and src/nserver.rs must be changed:
  - In src/nproxy.rs, on line 340, [lwd_uri_test] must be changed to [lwd_uri_main].
  - In src/nserver.rs, on line 85, [zproxy_uri] must be changed to [lwd_uri_main].
2) Run `$ cargo run --bin nserver`
3) Copy nym address displayed
4) Run `$ cargo run --bin nproxy "nserver address copied"`
5) Run `$ cargo run --release --package zingo-cli -- --chain "mainnet" --server "127.0.0.1:8080" --data-dir ~/wallets/mainnet_wallet`

