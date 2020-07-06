# Websocket Trading API Draft

### General

*<u>Note:</u> authentication request must be sent before any other trading request.*

Each request has the following structure:

```json
{
    "id":2,
    "method":"methodName",
    "params":{
        "desc": "params field could be either an object or an array"
    }
}
```

Every response has the following structure in case of success:

```json
{
    "id":2,
    "status":"ok"
} 
```

And in case of error response would be like:

```json
{
  "id":2,
  "status": "error",
  "code": 2222,
  "msg": "error description"
}
```

<u>Note</u>: the error will also be returned in case of system maintenance and absence of data for the response. 

Value of the `clOrdId` is assigned by the trader. It should be a unique string for each order. This string can be any length, but only the first 16 bytes are used by the exchange. If the length is less than 16 bytes the trading engine will add zero bytes to make it 16 bytes total.

Possible value of order's `status`:  `PENDING`, `ACCEPTED`, `REJECTED`, `CANCELED`, `FILLED`, `PARTIAL`, `TERMINATED`, `EXPIRED`, `TRIGGERED`.

Possible values of `ordType`: `MARKET`, `LIMIT`.

Possible values of `timeInForce`: `GTD`, `GTC`, `GTF`, `IOC`, `FOK`.

Possible values of `side`: `BUY`, `SELL`.

Possible values of `ch` (channel name): `error`, `orderStatus`, `orderFilled`, `orderCancelled`, `contractClosed`, `traderStatus`,`traderBalance`, `position`, `funding`, `leverage`.

For `BTCUSD-PERP`: order price should be positive and a <u>multiple of 5</u>, order quantity should be positive and <u>integral</u>.

All timestamps are provided in milliseconds.

------

### Trade Flow

------

#### Authentication

To authenticate, trader need to send:

```json
{
    "id":2,
    "method":"auth",
    "params":{
        "type":"token",
        "value":"yourToken"
    }
}
```

The server will respond with an appropriate `status` value.

If provided token isn't valid trader will receive the following message:

```json
{
    "ch":"error",
    "data":{
        "code":10501,
        "msg":"invalid credentials"
    }
}
```

Trader need to send `auth` message with the token every time he/she gets this error.

------

#### Place Order

To place a new order the following message should be sent:

```json
{
    "id":3,
    "method":"placeOrder",
    "params":{
        "symbol":"BTCUSD-PERP",
        "clOrdId":"7b17f2d9d94a477a",
        "ordType":"MARKET",
        "timeInForce":"IOC",
        "side":"BUY",
        "px": 0,
        "qty":25
    }
} 
```

Note: in case of `MARKET` order field `px` could be omitted.

The order can be either accepted or rejected by the exchange.

If the order has been accepted by the trading engine trader will receive the following message:

```json
{
    "ch":"orderStatus",
    "data":{
        "symbol":"BTCUSD-PERP",
        "timestamp":1594045833043,
        "clOrdId":"7b17f2d9d94a477a",
        "origClOrdId":"7b17f2d9d94a477a",
        "openTime":1594045833043,
        "orderStatus":"ACCEPTED",
        "orderType":"MARKET",
        "orderSide":"BUY",
        "timeInForce":"IOC",
        "px":0,
        "paidPx":462.25,
        "qty":25,
        "origQty":25,
        "leverage":20,
        "traderBalance":100665.635,
        "orderMargin":0,
        "positionMargin":231.125,
        "upnl":0,
        "pnl":468.16,
        "accumQty":125,
        "buyOrderMargin":0,
        "sellOrderMargin":0,
        "buyOrderQty":0,
        "sellOrderQty":0,
        "markPx":9239.7181,
        "volume":0
    }
}
```

The value of `clOrdId` can be used to cancel this order in the future (in case of `LIMIT` order).

------

#### Order Fills

When order is filled trader will receive the following message:

```json
{
    "ch":"orderFilled",
    "data":{
        "symbol":"BTCUSD-PERP",
        "timestamp":1594045833043,
        "clOrdId":"7b17f2d9d94a477a",
        "orderStatus":"FILLED",
        "newClOrdId":"\\)\ufffdV\"\ufffdMԶ\ufffdu^\ufffdj^V",
        "origClOrdId":"7b17f2d9d94a477a",
        "openTime":1594045833043,
        "px":0,
        "paidPx":0,
        "qty":0,
        "droppedQty":0,
        "origQty":25,
        "volume":0,
        "traderBalance":100665.635,
        "orderMargin":0,
        "positionMargin":231.125,
        "upnl":0,
        "pnl":468.16,
        "accumQty":125,
        "positionContracts":25,
        "positionVolume":231125,
        "positionLiquidationVolume":225375,
        "positionBankruptcyVolume":219568.75,
        "positionType":"LONG",
        "lastTradePx":9245,
        "lastTradeQty":25,
        "lastTradeTimestamp":1594045833043,
        "buyOrderMargin":0,
        "sellOrderMargin":0,
        "buyOrderQty":0,
        "sellOrderQty":0,
        "markPx":9239.7181,
        "contracts":[
            {
                "timestamp":1594045833043,
                "traderId":94889,
                "contractId":612705754,
                "origContractId":612705754,
                "openTime":1594045833043,
                "positionType":"LONG",
                "px":9245,
                "paidPx":462.25,
                "liquidationPx":9015,
                "bankruptcyPx":8782.75,
                "qty":25,
                "exitPx":0,
                "leverage":20,
                "oldClOrdId":"7b17f2d9d94a477a",
                "isIncrease":1,
                "entryQty":25,
                "exitQty":0,
                "exitVolume":0,
                "fundingPaidPx":0,
                "fundingQty":0,
                "fundingVolume":0,
                "fundingCount":0
            }
        ],
        "marketTrades":[
            {
                "orderPosition":"LONG",
                "px":9245,
                "paidPx":462.25,
                "qty":25,
                "leverage":20,
                "isMaker":0
            }
        ]
    }
}
```

The field `contracts` contains a list of contacts that were added to trader's position. 

Trader can use the value of `contractId` to close specific contract in the future by this ID.

Note: the value of `newClOrdId` is generated by the exchange. It's the ID of `FILLED` order and this order has the reference to the originally placed order via `origClOrdId`.

------

#### Order Cancellation

Specific order can be cancelled by its `clOrdId` via the following message:

```json
{
    "id":4,
    "method":"cancelOrder",
    "params":{
        "symbol":"BTCUSD-PERP",
        "clOrdId":"dff7de23224c4929"
    }
}
```

More than one order can be cancelled using the following message:

```json
{
    "id":5,
    "method":"cancelAllOrders",
    "params":{
        "symbol":"BTCUSD-PERP",
        "px":0,
        "side":"BUY"
    }
}
```

Trader can cancel all the orders (`side` and `px` are omitted) or just orders with the specified `side`  and/or `px`.

The following message will be received as a result of order/orders cancellation:

```json
{
    "ch":"orderCancelled",
    "data":{
        "symbol":"BTCUSD-PERP",
        "timestamp":1594048959068,
        "orderStatus":"CANCELLED",
        "orders":[
            {
                "timestamp":1594048949070,
                "clOrdId":"\ufffdEy\ufffd\n\ufffdI\u001e\ufffd\ufffdƕ\ufffd\ufffd\ufffdR",
                "oldClOrdId":"af0574c0f09743fe",
                "origClOrdId":"af0574c0f09743fe",
                "traderId":94889,
                "openTime":1594048949070,
                "orderType":"LIMIT",
                "orderSide":"BUY",
                "timeInForce":"GTC",
                "px":9100,
                "qty":85,
                "origQty":85,
                "paidPx":455,
                "leverage":20,
                "volume":0,
                "isClosing":0,
                "mayIncrease":0
            },
            {
                "timestamp":1594048949068,
                "clOrdId":"\ufffdiV\ufffd$3K7\ufffd\ufffd\u0010\ufffd \ufffd\ufffd\ufffd",
                "oldClOrdId":"36cbaf53b17c4a7e",
                "origClOrdId":"36cbaf53b17c4a7e",
                "traderId":94889,
                "openTime":1594048949068,
                "orderType":"LIMIT",
                "orderSide":"BUY",
                "timeInForce":"GTC",
                "px":9150,
                "qty":75,
                "origQty":75,
                "paidPx":457.5,
                "leverage":20,
                "volume":0,
                "isClosing":0,
                "mayIncrease":0
            }
        ],
        "traderBalance":100665.635,
        "orderMargin":0,
        "positionMargin":231.125,
        "upnl":15,
        "pnl":468.16,
        "accumQty":125,
        "buyOrderMargin":0,
        "sellOrderMargin":0,
        "buyOrderQty":0,
        "sellOrderQty":0,
        "markPx":9274.7042
    }
}
```

Note: `CENCELLED` order ID - `clOrdId`- differs from ID of placed order.

Field `orders` contains all the orders that have been cancelled. 

------

#### Close Contracts

##### One Contract

Trader can close specific contract using its ID (`contractId`). 

There is an option to specify order type (`MARKET` or `LIMIT`), price (only for `LIMIT`) and quantity (to close only a part of the contract) for contract close operation.

A particular contract can be closed via the following message:

```json
{
    "id":7,
    "method":"closeContract",
    "params":{
        "symbol":"BTCUSD-PERP",
        "positionId":612705754,
        "ordType":"MARKET",
        "px":0,
        "qty":10
    }
}
```

And the response message will be he following:

```json
{
    "ch":"contractClosed",
    "data":{
        "symbol":"BTCUSD-PERP",
        "orderIds":[
            "\ufffd\u000ef\u0010\ufffd\ufffdJN\ufffd\u0026\ufffd\ufffd\u001e\ufffd'!"
        ]
    }
}
```

The field `orderIds` contains an array of `clOrdId`s which have been created by the exchange to close the contract. Trader also will receive `orderStatus` and `orderFilled` messages related to these orders. The latter one will contain the information about affected contracts.

Note: if trader closes only a part of the contract the exchange will generate new ID for remained part of the contract (this ID can be found in `orderFilled` message).

##### All Contracts

The trader can close the position (all available contracts) via the following message:

```json
{
    "id":8,
    "method":"closePosition",
    "params":{
        "symbol":"BTCUSD-PERP",
        "ordType":"MARKET",
        "px":0
    }
}
```

Trader can specify `ordType` (`MARKET` or `LIMIT`) and `px` (if `ordType` is set to `LIMIT`).

A sequence of messages that will be received as a result is the same as in the case of closing a single contract (`contractClosed`, `orderStatus`, `orderFilled`).

------

#### Request Trader Status

Trader can request its status via the following message:

```json
{
    "id":9,
    "method":"getTraderStatus",
    "params":{
        "symbol":"BTCUSD-PERP"
    }
}
```

The exchange will send in response the following:

```json
{
    "ch":"traderStatus",
    "data":{
        "symbol":"BTCUSD-PERP",
        "traderBalance":100693.356,
        "orderMargin":465.5,
        "positionMargin":466,
        "upnl":0,
        "pnl":20.721,
        "accumQty":65,
        "markPx":9317.4806,
        "positionContracts":50,
        "positionVolume":466000,
        "positionLiquidationVolume":454500,
        "positionBankruptcyVolume":442700,
        "positionType":"LONG",
        "lastTradePx":9320,
        "lastTradeQty":1287,
        "lastTradeTimestamp":1594052957290,
        "buyOrderMargin":465.5,
        "sellOrderMargin":0,
        "contracts":[
            {
                "timestamp":1594052943135,
                "traderId":94889,
                "contractId":612835558,
                "openTime":1594052943135,
                "positionType":"LONG",
                "px":9320,
                "paidPx":466,
                "liquidationPx":9090,
                "bankruptcyPx":8854,
                "qty":50,
                "exitPx":0,
                "leverage":20,
                "entryQty":50,
                "exitQty":0,
                "exitVolume":0,
                "fundingPaidPx":0,
                "fundingQty":0,
                "fundingVolume":0,
                "fundingCount":0,
                "origContractId":612835558
            }
        ],
        "activeOrders":[
            {
                "clOrdId":"M\ufffdrC\u003c\ufffd\ufffd\ufffd\ufffd\u0005)5\ufffd",
                "origClOrdId":"M\ufffdrC\u003c\ufffd\ufffd\ufffd\ufffd\u0005)5\ufffd",
                "timestamp":1594052913550,
                "openTime":1594052913550,
                "orderType":"LIMIT",
                "orderSide":"BUY",
                "timeInForce":"GTC",
                "px":9310,
                "paidPx":465.5,
                "qty":50,
                "origQty":50,
                "leverage":20,
                "volume":0,
                "isClosing":0,
                "mayIncrease":0
            }
        ],
        "leverage":20,
        "buyOrderQty":50,
        "sellOrderQty":0
    }
}
```

This message contains trader's balance, current position, active orders (can be cancelled using corresponding `clOrdId`) and contracts (can be closed using corresponding `contractId`).

------

#### Change Leverage

Trader can change the value of current leverage via the  following message:

```json
{
    "id":2,
    "method":"changeLeverageAll",
    "params":{
        "symbol":"BTCUSD-PERP",
        "leverage":10
    }
}
```

Where `leverage` is the desired leverage.

The exchange will respond with the following:

```json
{
    "ch":"leverage",
    "data":{
        "symbol":"BTCUSD-PERP",
        "leverage":10,
        "contracts":[
            {
                "timestamp":1594053698798,
                "traderId":94889,
                "contractId":612847804,
                "oldContractId":612837668,
                "openTime":1594053061352,
                "positionType":"LONG",
                "px":9310,
                "paidPx":931,
                "liquidationPx":8845,
                "bankruptcyPx":8379,
                "qty":50,
                "exitPx":0,
                "leverage":10,
                "entryQty":50,
                "exitQty":0,
                "exitVolume":0,
                "fundingPaidPx":0,
                "fundingQty":0,
                "fundingVolume":0,
                "fundingCount":0,
                "origContractId":612837668
            },
            {
                "timestamp":1594053698798,
                "traderId":94889,
                "contractId":612847805,
                "origContractId":612835558,
                "openTime":1594052943135,
                "positionType":"LONG",
                "px":9320,
                "paidPx":932,
                "liquidationPx":8855,
                "bankruptcyPx":8388,
                "qty":50,
                "exitPx":0,
                "leverage":10,
                "oldContractId":612835558,
                "entryQty":50,
                "exitQty":0,
                "exitVolume":0,
                "fundingPaidPx":0,
                "fundingQty":0,
                "fundingVolume":0,
                "fundingCount":0
            }
        ],
        "activeOrders":[],
        "traderBalance":100693.356,
        "orderMargin":0,
        "positionMargin":1863,
        "upnl":-10,
        "pnl":20.721,
        "accumQty":115,
        "positionContracts":100,
        "positionVolume":931500,
        "positionLiquidationVolume":885000,
        "positionBankruptcyVolume":838350,
        "positionType":"LONG",
        "buyOrderMargin":0,
        "sellOrderMargin":0,
        "lastTradePx":9310,
        "lastTradeQty":866,
        "lastTradeTimestamp":1594053698097,
        "buyOrderQty":0,
        "sellOrderQty":0
    }
}
```

This message contains trader's balance, current position, active orders (can be cancelled using corresponding `clOrdId`) and contracts (can be closed using corresponding `contractId`) according to new leverage value.

------

#### Funding

Trader will receive the following message if he/she has open position during funding:

```json
{
    "ch":"funding",
    "data":{
        "symbol":"BTCUSD-PERP",
        "contracts":[
            {
                "timestamp":1594053698798,
                "traderId":94889,
                "contractId":612847804,
                "oldContractId":612837668,
                "openTime":1594053061352,
                "positionType":"LONG",
                "px":9310,
                "paidPx":931,
                "liquidationPx":8845,
                "bankruptcyPx":8379,
                "qty":50,
                "exitPx":0,
                "leverage":10,
                "entryQty":50,
                "exitQty":0,
                "exitVolume":0,
                "fundingPaidPx":0,
                "fundingQty":0,
                "fundingVolume":0,
                "fundingCount":0,
                "origContractId":612837668
            }
        ],
        "traderBalance":100693.356,
        "orderMargin":0,
        "positionMargin":1863,
        "upnl":-10,
        "pnl":20.721,
        "accumQty":100,
        "positionContracts":50,
        "positionVolume":931500,
        "positionLiquidationVolume":885000,
        "positionBankruptcyVolume":838350,
        "positionType":"LONG",
        "buyOrderMargin":0,
        "sellOrderMargin":0,
        "payout": 1.86,
        "payoutPerContract": 0.0186,
        "lastTradePx":9310,
        "lastTradeQty":866,
        "lastTradeTimestamp":1594053698097,
        "markPx": 9300,
        "positionMarginChange": 0,
        "buyOrderQty":0,
        "sellOrderQty":0
    }
}
```

------

#### Position

If trader's position has been liquidated and/or active orders have been terminated by the exchange the following message will be received:

```json
{
    "ch":"position",
    "data":{
        "symbol":"BTCUSD-PERP",
        "liquidatedContracts":[array of contracts],
        "terminatedOrders":[only clOrdId of orders],
        "traderBalance": 2000,
        "orderMargin": 0,
        "positionMargin": 0,
        "upnl": 0,
        "pnl": 0,
        "accumQty": 0,
        "positionContracts": 0,
        "positionVolume": 0,
        "positionLiquidationVolume": 0,
        "positionBankruptcyVolume": 0,
        "lastTradePx": 9300,
        "lastTradeQty": 100,
        "lastTradeTimestamp": 1594053698097,
        "buyOrderMargin": 0,
        "sellOrderMargin": 0,
        "traderBalanceIncrement": 1000,
        "buyOrderQty": 0,
        "sellOrderQty": 0,
        "markPx": 9285
    }
}
        
```

Field `liquidatedContracts` contains an array of liquidated contracts (the same structure as in `traderStatus` message).

------

#### Errors

Exchange can respond with an error to incoming trade message:

```json
{
    "ch":"error",
    "data":{
        "symbol":"BTCUSD-PERP",
        "code":10,
        "msg":"ID doesn't exist"
    }
}
```

------
