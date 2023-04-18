---
title: Prodigy API Reference

language_tabs: # must be one of https://github.com/rouge-ruby/rouge/wiki/List-of-supported-languages-and-lexers
- plaintext

toc_footers:
- <a href='https://prodigy.fi'>Prodigy Protocol</a>

includes:


search: true

code_clipboard: true

meta:
- name: description
- content: Documentation for the Prodigy Protocol API
---

# Prodigy API Documentation

## <b> Overview </b>

This document describes the API for the Prodigy Protocol platform.

By using any API provided by Prodigy, you agree to its Terms of Use and Privacy Policy.

## Currently Supported Instruments
* BTC-USD: Bitcoin/USD spot price

# Websocket API

## Connectivity (alpha)
wss://api.prodigy.fi:36821

## Authorization
Please contact your Prodigy representative to request an Alpha CLOB token for your connectivity.

No additional authorization is required at this time.

After receiving your token, ensure that it is placed in the token field of your subscription request.

Token example with characters rewritten: `0xdaaf5b74xxx8afd78xxxxx2eb5db442edb53d760ea16xxxc67707xxxxxc925e4`

## Websocket Availability

* Sunday evening to Saturday evening EST.
* Currently includes a maintenance window on Thursday afternoons.
* Alpha environment times are not guaranteed.

## Websocket FAQ
Q. I am getting an ERROR: `token wasn't supplied`?
A. Token value must be supplied with the ws subscribeMessage for authentication.

Q. Why is the price of BTC 10x more than it should be and what is the “denominator” value in the response?
A. The denominator value is the factor to which we need to divide price by to derive the USD value of the asset. For example:

if "BestBidPrice" :"242385" , "Denominator":"10" & "Symbol":"BTC-USD" then the
USD price of BTC is $24,238.5

Q. What are the different responses coming from the depth ws connection?
A. Aside from confirmation of the connection being established, the initial depth data message will detail the current depth of the market at each price level on both the buy and sell side. Subsequent messages will update depth at those price levels based on actions taken by market participants. These actions can be new orders being added, existing orders being canceled, modified, or taken.

## Depth of Market subscription (depth)

Gets depth data for a market.

### Subscribe message

> Subscribe message

```plaintext
{
'user': 'g',
'action': 'subscribe',
'type': 'depth',
'value': ['BTC-USD'],
"token":"<provided by Prodigy>"
};

```

### Response
The initial depth data response returns information about the market and its current buy and sell levels with data that describes the depth at each level.

Data | Definition
--------- |------------
Denominator | Factor to divide <Price> by for USD value      
DisplaySymbol | Display symbol of market
SecurityID | Exchange ID for market       
Symbol | Symbol of market       
buy_levels | Array of buy orders by level (details for each below)       
&nbsp;&nbsp;&nbsp;NumOfOrders | Number of orders at this buy level        
&nbsp;&nbsp;&nbsp;Price | Price of orders at this buy level       
&nbsp;&nbsp;&nbsp;Qty | Quantity of orders at this buy level       
sell_levels | Array of sell orders by level (details for each below)       
&nbsp;&nbsp;&nbsp;NumOfOrders | Number of orders at this sell level        
&nbsp;&nbsp;&nbsp;Qty | Price of orders at this sell level       
&nbsp;&nbsp;&nbsp;sell_levels | Quantity of orders at this sell level  


Example response:

>Example response

```plaintext
{
  "data": {
"Denominator":"100",
"DisplaySymbol":"BTC-USD",
"SecurityID":"1000000189",
"Symbol":"BTC-USD",
"buy_levels": [...
{
"NumOfOrders": "10",
"Price": "2037600",
"Qty": "4660000",
},
...],
"sell_levels": [...
{
"NumOfOrders": "7",
"Price": "2056150",
"Qty": "6370000",
},
...],
},
  "Type":3
```

### Subsequent Responses

Updates depth data for a market. Subsequently, depth data responses return updates to the depth of the market.

Data | Definition
--------- |------------
Denominator | Factor to divide <Price> by for USD value      
DisplaySymbol | Display symbol of market
SecurityID | Exchange ID for market       
Symbol | Symbol of market       
updates | Array of updates to market depth (details for each below)       
&nbsp;&nbsp;&nbsp;Action | Action that updated market depth (New, Delete, Add, Change)        
&nbsp;&nbsp;&nbsp;NumOfOrders | Number of orders that were updated       
&nbsp;&nbsp;&nbsp;Price | Price level of orders that were updated       
&nbsp;&nbsp;&nbsp;Qty | Quantity of securities that were updated       
&nbsp;&nbsp;&nbsp;Side | Side of the market of this update (Buy / Sell)         

Example subsequent message response:

> Example subsequent message response

```plaintext
{
  "data": {
"Denominator":"100",
"DisplaySymbol":"BTC-USD",
"SecurityID":"1000000189",
"Symbol":"BTC-USD",
"updates": [
{
"Action": "New",
"NumOfOrders": "1",
"Price": "2037900",
"Quantity": "500000",
"Side": "Buy",
},
{
"Action": "Change",
"NumOfOrders": "9",
"Price": "2056150",

"Quantity": "6160000",
"Side": "Sell",
}
],
  },
  "Type":4
}
```

## Price Info Subscription (price_info)
Gets price data for a market.

### Subscribe Message

> Subscribe message

```plaintext
{
'user': 'g',
'action': 'subscribe',
'type': 'price_info',
'value': ['BTC-USD'],
"token":"<provided by Prodigy>"
};
```

### Response

Data | Definition
--------- |------------
BestBidPrice  | Price level of best bid
BestBidQty  | Quantity available at best bid
BestOfferPrice  | Price level of best ask
BestOfferQty  | Quantity available at best ask
Denominator  | Factor to divide <Price> by for USD value
LastTradePrice  | Price that last trade executed
LastTradeQty  | Quantity of last trade
LastTradeTimestamp  | Linux timestamp of last trade
Symbol  | Symbol of market


Example message response:

> Example message response

```plaintext
{
"data": {
"BestBidPrice":"242385",
"BestBidQty":"59300",
"BestOfferPrice":"242390",
"BestOfferQty":"200",
"Denominator":"10",
"LastTradePrice":"242385",
"LastTradeQty":"97",
"LastTradeTimestamp":"1660498301",
"Symbol":"BTC-USD"
},
"Type":5
}
```



# stunnel Connectivity
Prodigy uses stunnel, an open-source implementation used widely as an SSL tunneling tool. Refer to http://www.stunnel.org for more information.

A stunnel provides a secure way to send FIX messages from your FIX client, over the internet, to Prodigy. The possibilities of implementing this will vary based on your existing FIX client setup, but in general it is simple:

FIX Client -> Client-side stunnel -> INTERNET -> Exchange-side stunnel -> <PLACE_HOLDER>

Below is a guide to using a dockerized stunnel implementation available to you from Prodigy.

## Setup a client-side stunnel to connect to Prodigy’s CLOB

### Important environment variables

`export CLOB_HOST=clob-alpha.prodigy`

`export STUNNEL_PORT=[THE_LOCAL_PORT_TO_OPEN]`

<aside class="notice">
NOTE: (Optional) If TCP port 36814 is in use, change the local tunnel port to an available one.
</aside>

Alternatively, these environment variables can be passed in on the command line (see below).

### Download Prodigy stunnel docker-compose.yml file

> Prodigy stunnel docker-compose.yml file

```plaintext
version: '3.9'
services:
stunnel:
``#image: dweomer/stunnel``
image: gitlab.dev.blockriver.tech:5050/gyeu/stunnel
container_name: stunnel
ports:
- ${STUNNEL_PORT:-36814}:${STUNNEL_PORT:-36814}
  environment:
- STUNNEL_SERVICE=fixs-client
- STUNNEL_CLIENT=yes
- STUNNEL_ACCEPT=${STUNNEL_PORT:-36814}
- STUNNEL_CONNECT=${CLOB_HOST:-localhost}:36813
- STUNNEL_PSKSECRETS=/etc/stunnel/psk.txt
  volumes:
- ./psk.dev.txt:/etc/stunnel/psk.txt:ro
  logging:
  driver: "json-file"
  options:
  max-size: "50m"
  max-file: "10"
```


### Build Docker container with the Prodigy stunnel docker-compose.yml file

`cd to/dir/that/contains/docker-compose.yml`

`docker-compose up docker-compose.yml`

This will keep the container running in the foreground and we can press ctrl-c at any time to stop it.

### Install docker and docker-compose

We advise installing Docker Desktop. While you can install Docker and Docker Compose, Docker Desktop is an easy-to-install application for your macOS, Linux, or Windows environment that enables you to build and share containerized applications and microservices. Installing Docker Desktop also includes docker-compose out of the box, along with Docker engine and Docker CLI. Installing Docker Desktop is the fastest and simplest way to get started.

Alternatively, if you prefer to run the container in the background, use the following command:

`docker-compose up -d docker-compose.yml`

If running the container in the background, remember to stop the stunnel container when not using it, as it will occupy the port until the container is terminated.

Update configuration of your FIX client

* target host = 127.0.0.1
* target port = 36814


# FIX API

FIX (Financial Information eXchange) API is a messaging protocol used in the electronic trading industry. It allows clients and brokers to enter orders, submit cancel requests, and receive fills. The Prodigy Protocol FIX API is intended to be used in conjunction with the FIX 4.4 Protocol Specification.

To establish a connection using the FIX protocol, clients and brokers use a software called FIX engines. At a predetermined start time, Client A and Broker B connect their FIX engines using a predetermined host and comp ID.

Authentication results over a secure FIX protocol ensuring privacy of data exchange. The authentication process provides privacy and a reliable FIX protocol for effective communication between clients and brokers throughout electronic trade.  Responses are formatted as strings and given numerical tags for efficient communication between clients and brokers.

In all of the following example messages, the “Anatomy of” section displays the FIX tags in header, body, trailer format.

## `Logon <A>`

Use the `Logon <A>` message to authenticate a user establishing a connection to a remote system. The `Logon <A>` message must be the first message sent by the application requesting to initiate a FIX session.

The `HeartBtInt <108>` field is used to declare the timeout interval for generating heartbeats,  with both sides holding the same value. The HeartBtInt <108> value should be agreed upon by the two firms, specified by the Logon initiator, and echoed back by the Logon acceptor.

Upon receipt of a `Logon <A>` message, the session acceptor will authenticate the party requesting connection and issue a `Logon <A>` message as acknowledgment that the connection request has been accepted. The acknowledgment `Logon <A>` can also be used by the initiator to validate that the connection was established with the correct party.

The session acceptor must be prepared to immediately begin processing messages after receipt of the `Logon <A>`. The session initiator can choose to begin transmission of FIX messages before receipt of the confirmation `Logon <A>`, however it is recommended to wait for the return `Logon <A>` message to accommodate encryption key negotiation.

The confirmation `Logon <A>` can be used for encryption key negotiation. If a session key is deemed to be weak, a stronger session key can be suggested by returning a `Logon <A>` message with a new key. This is only valid for encryption protocols that allow for key negotiation. (See the FIX Web Site's Application notes for more information on a method for encryption and key passing.)

The `Logon <A>` message can be used to specify the <code>MaxMessageSize <383></code> supported. This can be used to control fragmentation rules for very large messages which support fragmentation. It can also be used to specify the MsgTypes supported for both sending and receiving.

### Anatomy of a Logon Message

Payload example

> Logon payload example

```plaintext
8=FIX.4.4 | 9=94 | 35=A | 49=ACCOUNT_MNEMONIC | 56=order_router | 34=1 | 52=20221007-15:49:42.769 | 98=0 | 108=30 | 141=Y | 10=112 |
```

Where:

| Field            | Value             |  Tag Name   | Is Required | Definition                                                                                                                                                                                  |
|------------------|-------------------|-----|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <MessageHeader>  |                   |     | Y           | MsgType<35>=A                                                                                                                                                                               |
| 8                | FIX 4.4           | BeginString    | Y           | FIX protocol version                                                                                                                                                                        |
| 9                | 94                |  BodyLength   | Y           | Message length in bytes                                                                                                                                                                     |
| 35               | A                 |   MsgType  | Y           | Message type is a Logon message                                                                                                                                                             |
| 49               | ACCOUNT_MNEMONIC  |  SenderCompID   | Y           | ID of party sending message                                                                                                                                                                 |
| 56               | order_router      |  TargetCompID   | Y           | ID of party receiving message                                                                                                                                                               |
| 34               | 1                 |  MsgSeqNum   | Y           | Integer message sequence number                                                                                                                                                             |
| 52               | 20221007-14:49.769 | SendingTime    | Y           | Time of message                                                                                                                                                                             |
| 98               | 0                 |  EncryptMethod   | Y           | Method of encryption                                                                                                                                                                        |
| 108              | 30                |  HeartBint   | Y           | 30 second interval between heartbeat messages                                                                                                                                               |
| 141              | Y                 |  ResetSeqNumFlag | Y           | Indicates that both sides of the FIX session should restart sequence numbers                                                                                                                |
| 10               | 112               | CheckSum | Y           | Checksum of FIX message                                                                                                                                                                     |
| 95               | RawData           | RawDataLength              | N           | Required for some authentication methods                                                                                                                                                    |
| 96               | data            | RawData         | N           | Required for some authentication methods                                                                                                                                                    |
| 789              |                   |  NextExpectedMsgSeqNum        | N           | Optional, alternative via counterparty bi-lateral agreement message gap detection and recovery approach                                                                                     |
| 383              |                   | MaxMessageSize          | N           | Can be used to specify the maximum number of bytes supported for messages received                                                                                                          |
| 384              |                   |        NoMsgTypes          | N           | Specifies the number of repeating `RefMsgTypes <372>` specified                                                                                                                             |
| => 372           |                   |     RefMsgType             | N           | Specifies a specific, supported `MsgType <35>`. Required if `NoMsgTypes <384>` is > 0. Should be specified from the point of view of the sender of the `Logon <A>` message.                 |
| => 385           |                   | MsgDirection                | N           | Indicates direction (send vs. receive) of a supported `MsgType <35>`. Required if `NoMsgTypes <384>` is > 0. Should be specified from the point of view of the sender of the `Logon <A>` message. |
| 464              |                   | TestMessageindicator | N           | Can be used to specify that this FIX session will be sending and receiving 'test' vs. 'production' messages.                                                                                |
| 553              |                   | Username                    | N           |                                                                                                                                                                                             |
| 554              |                   | Password                    | N           | Note: minimal security exists without transport-level encryption.                                                                                                                                                                                            |
| `<MessageTrailer>` |                   |                             |             |                                                                                                                                                                                             |


## `Logout <5>`

Use the `Logout <5>` message to initiate or confirm the termination of a FIX session. If disconnection occurs without the exchange of Logout messages, this should be interpreted as an abnormal condition.

Before closing the session, the initiator of the Logout message should wait for the opposite side to respond with a confirming Logout message. This gives the remote end a chance to perform any necessary Gap Fill operations. If the remote side does not respond in an appropriate time frame, the session may be terminated.

After sending the Logout message, the initiator should not send any messages unless requested to do so by the logout acceptor via a `Resend Request <2>`.

### Anatomy of a Logout Message

Logout payload example

> Logout payload example

```plaintext
8=FIX.4.4 | 9=66 | 35=5 | 49=ACCOUNT_MNEMONIC | 52=20221027-00:00:00.949 | 56=order_router | 10=027 |
```

Where:

| Field           | Value                 |  Tag Name   | Is Required |   Definition  |
|-----------------|-----------------------|-----|-------------|-----|
| 8               | FIX 4.4               |   BeginString  | Y           | FIX protocol version    |
| 9               | 66                    | BodyLength    | Y           | Message length in bytes    |
| 35              | 5                     | MsgType    | Y           |  Message type is a Logout message   |
| 49              | ACCOUNT_MNEMONIC      |  SenderCompID   | Y           |  ID of party sending message   |
| 52              | 20221027-00:00:00.949 |   SendingTime  | Y           |  Time of message   |
| 56              | order_router          |  TargetCompID   | Y           |  ID of party receiving message   |
| 10              | 027                   | Checksum    | Y           |  Checksum of FIX message   |
| `<MessageHeader>`  |                       |                       |             |     |
| 58              | String                | Text    | N           |  Information to include with the message   |
| 354             |                       | EncodedTextLen    |             |  Must be set if Encoded Text <355> field is specified and must immediately precede it   |
| 355             |                       |  EncodedText   |             |  Encoded (non-ASCII characters) representation of the Text`<58>`field in the encoded format specified via the MessageEncoding `<347>` field
| `<MessageTrailer>`              |                       |     | Y           |     |

## `NewOrderSingle <D>`

Use the `New Order <D>` message type to electronically submit securities and forex orders to a broker for execution.

Orders can be submitted with special handling and execution instructions. Handling instructions refer to how the broker should handle the order on its trading floor (see `HandlInst <21>` field). Execution instructions contain explicit directions as to how the order should be executed (see `ExecInst <18>` field).

New Order messages received with the `PossResend <97>` flag set in the header should be validated by `ClOrdID <11>`. Implementations should also check order parameters (side, symbol, quantity, etc.) to determine if the order had been previously submitted. PossResends previously received should be acknowledged back to the client via an Execution - Status message. PossResends that were not previously received should be processed as a new order and acknowledged via an Execution - New message.

The value specified in the `TransactTime <60>` field should allow the receiver of the order to apply business rules to determine if the order is potentially "stale," for example, in the event that there have been communication problems.

The `OrdType <40>` field can be set to Market, Limit, Forex - Swap, or Previously Quoted, while the `Product <460>` field is set to 4 (CURRENCY).

To "take" an `IOI <6>` (or `Quote <S>`) from an ECN or exchange and not display the order on the book, the New Order message must contain the `TimeInForce <59>` field set to "ImmediateOrCancel" and an OrdType <40> field set to "Previously Indicated" ( or "Previously Quoted").
Anatomy of a NewOrderSingle Message


NewOrderSingle payload example

> NewOrderSingle payload example

```plaintext
8=FIX.4.4 | 9=66 | 35=5 | 49=ACCOUNT_MNEMONIC | 52=20221027-00:00:00.949 | 56=order_router | 10=027 |
```


Where:

| Field            | Value                 | Tag Name           | Is Required | Definition                                                                                                                             |
|------------------|-----------------------|--------------------|-------------|----------------------------------------------------------------------------------------------------------------------------------------|
| 8                | FIX 4.4               | BeginString        | Y           | FIX protocol version                                                                                                                   |
| 9                | 66                    | BodyLength         | Y           | Message length in bytes                                                                                                                |
| 35               | 5                     | MsgType            | Y           | Message type is a Logout message                                                                                                       |
| 49               | String                | ACCOUNT_MNEMONIC   | Y           | ID of party sending message                                                                                                            |
| 52               | 20221027-00:00:00.949 | SendingTime        | Y           | Time of message                                                                                                                        |
| 56               | order_router          | String             | Y           | ID of party receiving message                                                                                                          |
| 10               | 027                   | Checksum           | Y           | Checksum of FIX message                                                                                                                |
| `<MessageHeader>` |                       |                    | Y           | MsgType<35>=D                                                                                                                          |
| 11               | String                | ClOrdID            | Y           | Unique identifier of the order as assigned by institution or by the intermediary with closest association with the investor            |
| 1                | String                | Account            | N           | Account mnemonic as agreed between buy and sell sides, e.g. broker and institution or investor/intermediary and fund manager           |
| 100              | Exchange              | ExDestination      | N           | Execution destination as defined by institution when order is entered                                                                  |
| `<Instrument>`   |                       | Instrument fields  | Y           | One or more of the following instrument tags                                                                                           |
| _55_             | _String_                | _Symbol_           | _N_         | _Common and human readable representation of the securities_                                                                           |
| _48_             | _String_                   | _SecurityID_       | _N_         | _Takes precedence in identifying security to counterparty over SecurityA/tID <455> block. Requires SecurityIDSource <22> if specified_ |
| _22_             | _String_                   | _SecurityIDSource_ | _N_         | _Alternate identifier for this security_                                                                                               |
| 54               |  Side                     | char               | Y           | _Side of the order_                                                                                                                    |
| 60               |  TransactTime                   | UTCTimestamp       | Y           | Time this order request was initiated/released by the trader, trading system, or intermediary                                          |
| 38               |                       |                    | Y           | Quantity ordered                                                                                                                       |
| 40               |  OrdType                   | char               | Y           | Order Type`1 = market 2 = limit 3 = stop 4 = spot limit`                                                                               |
| 44               |  Price                    | Price              | N*          | Price per unit of quantity Required for limit orders                                                                                   |
| 59               |  TimeInForce                    | char               | N*          |   Specifies how long the order remains in effect. Absence of this field is interpreted as DAY                                                                                                                                     |
| `<MessageTrailer>`                 |                       | Y                  |             |                                                                                                                                        |


## `OrderCancelRequest <F>`

Use the `Order Cancel Request <F>` message request to cancel all of the remaining quantity of an existing order. Note that the `Order Cancel/Replace Request <G>` should be used to partially cancel or reduce an order.

The request will only be accepted if the order can successfully be pulled back from the exchange floor without executing.

A cancel request is assigned a `ClOrdID <11>` and is treated as a separate entity. If rejected, the `ClOrdID <11>` of the cancel request will be sent in the `Cancel Reject <9>` message, as well as the `ClOrdID <11>` of the actual order in the `OrigClOrdID <41>` field. The `ClOrdID <11>` assigned to the cancel request must be unique amongst the `ClOrdID <11>` assigned to regular orders and replacement orders.

### Anatomy of a OrderCancelRequest Message

OrderCancelRequest payload example

> OrderCancelRequest payload example

```plaintext
8=FIX.4.4 | 9=197 | 35=F | 49=EIB_TRADER1_EMAIL_COM | 56=order_router | 34=6 | 52=20221103-05:15:53.225 | 41=ACC_OO1-1567450006911 | 11=ACC_001_1667450006914 | 1=SCC_001 55=BTC-USD | 48=1000000189 | 54=1 | 60=20221103-04:33:27.139 | 38=69 | 10=238
```

Where: 

| Field           | Value                |  Tag Name   | Is Required |   Definition  |
|-----------------|----------------------|-----|-------------|-----|
| `<MessageHeader>` |                      |     | Y           | MsgType<35>=F    |
| 41              | ACC_001_1567450006911 | OrigIOrdID    | Y           |   ClOrdID <11> of the previous order (NOT the initial order of the day when canceling or replacing an order  |
| 11              | ACC_001_1667450006914 |  CIOrdiD   | Y           |  Unique ID of cancel request as assigned by the institution   |
| 1               | Value=ACC_001        |  Account   | N           |  Account mnemonic as agreed between buy and sell sides, e.g. broker and institution or investor/intermediary and fund manager   |
| 55              | Value=BTC-USD           |  Symbol   | Y           |  Common and human readable representation of the securities   |
| 48              | 1000000189                    |  SecurityiD   | N           |  Alternate security identifier   |
| 54              | 1                    |  Side   | Y           |  Side of the order   |
| 60              | 20221103-04:33:27.139 |   TransactTime  | Y           |   Time this order request was initiated/released by the trader, trading system, or intermediary  |
| 38              | 69                   |   OrderQty  | Y           |  Number of instruments ordered   |
| `<MessageTrailer>` |                      |     | Y           |     |


## `OrderMassStatus <AF>`

The `Order Mass Status Request <AF>` message requests the status of orders that match the criteria specified in the request.

A mass status request is assigned `ClOrdID <11>` and is treated as a separate entity. Upon processing the request, ExecutionReports with `ExecType <150>`="Order Status" are returned for all orders matching the criteria provided on the request.

Use the `MassStatusReqType <585>` field to specify the selection criteria for the orders to be included upon the request.

### Anatomy of a OrderMassStatus Message

OrderMassStatus payload example

> OrderMassStatus payload example

```plaintext
8=FIX.4.4 | 9=93 35=AF | 49=EIB_TRADER1_EMAIL_COM | 56=order_router | 34=5. | 52=20221103-05:15:40.884 | 584=4 | 585=7 | 1=* | 10=039
```

Where:

| Field                 | Value | Tag Name        | Is Required | Definition                                                     |
|-----------------------|-------|-----------------|-------------|----------------------------------------------------------------|
|      <MessageHeader>                 |       |                 | Y           | MsgType<35>=AF                                                 |
| 584                   | 4     | MassStatusReqID | Y           | Unique ID of mass status request as assigned by the institution |
| 585                   | 7     |   MassStatusReqType              | Y           | Specifies the scope of the mass status request                 |
| <B> Valid Values </B> |       |                 |             |                                                                |
|                       |       |                 | 1           | Status for orders for a security                               |
|                       |       |                 | 2           | Status for orders for an Underlying security                   |
|                       |       |                 | 3           | Status for orders for a Product`<460>`                         |
|                       |       |                 | 4           | Status for orders for a CFICode `<461>`                        |
|                       |       |                 | 5           | Status for orders for a SecurityType `<167>`                   |
|                       |       |                 | 6           | Status for orders for a trading session                        |
|                       |       |                 | 7           | Status for all orders                                                               |
|                       |       |                 | 8           | Status for orders for a PartyID <448>                                                             |
|                       |       |                 | 9           | Status for orders for an account                                                              |
| 1                     | *     |   Account              | N           |    Account                                                            |
| `<MessageTrailer>`    |       |                 | Y           |                                                                |


## `TradeCaptureReportRequest <AD>`

Use the `Trade Capture Report Request <AD>` to:
* Request one or more trade capture reports based on specific selection criteria provided on the trade capture report request
* Subscribe for trade capture reports based upon selection criteria provided on the trade capture report request.

Except for `TradeRequestID <568>` and `SubscriptionRequestType <263>`, each field in the Trade Capture Report Request <AD> identifies filters. After meeting all specified filters, trade reports will be returned. Note that the filters are combined using an implied "and" - a trade report must satisfy every specified filter to be returned.

After looking at the `TradeCaptureReportRequest` in the example section below, notice the response sent back to the client is an execution report that matches the filters and queries in the original TradeCaptureReportRequest.

### Anatomy of a `TradeCaptureReportRequest` Message

TradeCaptureReportRequest payload example

> TradeCaptureReportRequest payload example

```plaintext
8=FIX.4.4 | 9=117 | 35=AD | 34=3 | 49=EIB_TRADER1_EMAIL_COM | 52=20221103-05:15:40.884 | 56=order_router | 1=* | 58=* | 100=* | 568=T21666588915 | 569=0 | 10=137
```

Where:

| Field                 | Value       | Tag Name       | Is Required | Definition                                                                                                                                   |
|-----------------------|-------------|----------------|------------|----------------------------------------------------------------------------------------------------------------------------------------------|
|`<MessageHeader> `                 |             |                | Y          | MsgType<35>=AF                                                                                                                               |
| 1                     | *           | Account        | Y          | Account mnemonic as agreed between buy and sell sides                                                                                        |
| 58                    | *           | Text           | Y          | Free format text string                                                                                                                      |
| 100                   | *           | ExDestination        | Y          | Execution destination as defined by institution when order is entered                                                                        |
| 568                   | T21656588915 | TradeRequestID   | Y          | Identifier for the trade request                                                                                                             |
| 569                   | 0           | TradeRequestType   | Y          | Type of Trade Capture Report                                                                                                                 |
| <b> Valid Values </b> |                 |                                                                                                                                              |
|                       |        |        | 0          | all trades                                                                                                                                   |
|                       |             |                | 1          | Matched trades matching Criteria provided on request (parties, exec id, trade id, order id, instrument, input source, etc.)                                                                                                                                            |
|                       |       |            | 2          | Unmatched trades that match criteria                                                                                                      |
|                       |             |  | 3          | Unreported trades that match criteria                                                         |
|                       |             |    | 4          | Advisories that match criteria |
| <MessageTrailer>      |             |                | Y          |                                                                                                                                              |

