# Pricefeed.Service-Nibiru

#Làm nhiệm vụ nhận 200 điểm Nibiru

1/ Tải về bộ cài đặt pricefeeder:

    apt install curl -y
    curl -s https://get.nibiru.fi/pricefeeder! | bash
    
2/ Export thông tin: Lưu ý dùng 24 từ khoá của bạn:

    export FEEDER_MNEMONIC="dán 24 từ khoá vào đây"

    export CHAIN_ID="nibiru-itn-1"
    export GRPC_ENDPOINT="localhost:9090"
    export WEBSOCKET_ENDPOINT="ws://localhost:26657/websocket"
    export EXCHANGE_SYMBOLS_MAP='{ "bitfinex": { "ubtc:uusd": "tBTCUSD", "ueth:uusd": "tETHUSD", "uusdt:uusd": "tUSTUSD" }, "binance": { "ubtc:uusd": "BTCUSD", "ueth:uusd": "ETHUSD", "uusdt:uusd": "USDTUSD", "uusdc:uusd": "USDCUSD", "uatom:uusd": "ATOMUSD", "ubnb:uusd": "BNBUSD", "uavax:uusd": "AVAXUSD", "usol:uusd": "SOLUSD", "uada:uusd": "ADAUSD", "ubtc:unusd": "BTCUSD", "ueth:unusd": "ETHUSD", "uusdt:unusd": "USDTUSD", "uusdc:unusd": "USDCUSD", "uatom:unusd": "ATOMUSD", "ubnb:unusd": "BNBUSD", "uavax:unusd": "AVAXUSD", "usol:unusd": "SOLUSD", "uada:unusd": "ADAUSD" } }'
    export VALIDATOR_ADDRESS=$(nibid keys show wallet --bech val -a)
    
3/ Tạo hệ thống:

    sudo tee /etc/systemd/system/pricefeeder.service<<EOF
    [Unit]
    Description=Nibiru Pricefeeder
    Requires=network-online.target
    After=network-online.target

    [Service]
    Type=exec
    User=$USER
    Group=$USER
    ExecStart=/usr/local/bin/pricefeeder
    Restart=on-failure
    ExecReload=/bin/kill -HUP $MAINPID
    KillSignal=SIGTERM
    PermissionsStartOnly=true
    LimitNOFILE=65535
    Environment=CHAIN_ID='$CHAIN_ID'
    Environment=GRPC_ENDPOINT='$GRPC_ENDPOINT'
    Environment=WEBSOCKET_ENDPOINT='$WEBSOCKET_ENDPOINT'
    Environment=EXCHANGE_SYMBOLS_MAP='$EXCHANGE_SYMBOLS_MAP'
    Environment=FEEDER_MNEMONIC='$FEEDER_MNEMONIC'
    Environment=VALIDATOR_ADDRESS='$VALIDATOR_ADDRESS'
    [Install]
    WantedBy=multi-user.target
    EOF
    
 4/ Chạy lệnh:
 
    nibid tx oracle set-feeder $(nibid keys show wallet -a) --from wallet -y
    
 5/ Khởi đông hệ thống kiểm tra:
 
    sudo systemctl daemon-reload && \
    systemctl restart systemd-journald.service && \
    sudo systemctl enable pricefeeder && \
    sudo systemctl start pricefeeder
   
 6/ Check logs:
 
    sudo journalctl -fu pricefeeder -o cat
   
