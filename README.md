# Erlang MQTT Client [![Build Status](https://travis-ci.org/emqtt/emqttc.svg?branch=master)](https://travis-ci.org/emqtt/emqttc)

emqttc support MQTT V3.1/V3.1.1 Protocol Specification, and support parallel connections and auto reconnect to broker.

emqttc requires Erlang R17+.

## Features

Both MQTT V3.1/V3.1.1 Protocol Support

QoS0, QoS1, QoS2 Publish and Subscribe

SSL Socket Support

## Usage

### simple 

examples/simple_example.erl

```erlang
%% connect to broker
{ok, C} = emqttc:start_link([{host, "localhost"}, {client_id, <<"simpleClient">>}]),

%% subscribe
emqttc:subscribe(C, <<"TopicA">>, 0),

%% publish
emqttc:publish(C, <<"TopicA">>, <<"Payload...">>),

%% receive message
receive
    {publish, Topic, Payload} ->
        io:format("Message Received from ~s: ~p~n", [Topic, Payload])
after
    1000 ->
        io:format("Error: receive timeout!~n")
end,

%% disconnect from broker
emqttc:disconnect(C).

```

### gen_server

examples/gen_server_example.erl

```erlang

...

%% Connect to broker when init
init(_Args) ->
    {ok, C} = emqttc:start_link([{host, "localhost"},
                                 {client_id, <<"simpleClient">>},
                                 {logger, info}]),
    emqttc:subscribe(C, <<"TopicA">>, 1),
    self() ! publish,
    {ok, #state{mqttc = C, seq = 1}}.

%% Receive Publish Message from TopicA...
handle_info({publish, Topic, Payload}, State) ->
    io:format("Message from ~s: ~p~n", [Topic, Payload]),
    {noreply, State};

```

## Build

```

$ make

```

## Connect to Broker

Connect to MQTT Broker:

```erlang

{ok, C1} = emqttc:start_link([{host, "test.mosquitto.org"}]).

%% with name 'emqttclient'
{ok, C2} = emqttc:start_link(emqttclient, [{host, "test.mosquitto.org"}]).

```
### Connect Options

```erlang

-type mqttc_opt() :: {host, inet:ip_address() | binary() | string()}
                   | {port, inet:port_number()}
                   | {client_id, binary()}
                   | {clean_sess, boolean()}
                   | {keepalive, non_neg_integer()}
                   | {proto_ver, mqtt_vsn()}
                   | {username, binary()}
                   | {password, binary()}
                   | {will, list(tuple())}
                   | ssl
                   | {logger, atom() | {atom(), atom()}}
                   | {reconnect, non_neg_integer() | {non_neg_integer(), non_neg_integer()} | false}.
```

Option | Value | Default | Description | Example
-------|-------|---------|-------------|---------
host   | inet:ip_address() or string() | "locahost" | Broker Address | "locahost"
port   | inet:port_number() | 1883 | Broker Port | 
client_id | binary() | random clientId | MQTT ClientId | <<"slimpleClientId">>
clean_sess | boolean() | true | MQTT CleanSession | 
keepalive | non_neg_integer() | 60 | MQTT KeepAlive(secs) 
proto_ver | mqtt_vsn()			| 4 | MQTT Protocol Version | 3,4
username | binary()
password | binary()
will | list(tuple()) | undefined | MQTT Will Message | [{qos, 1}, {retain, false}, {topic, <<"WillTopic">>}, {payload, <<"I die">>}]
ssl  | | | ssl socket | 
logger | atom() or {atom(), atom()} | info | Client Logger | error, {opt, info}, {lager, error}
reconnect | false, or integer() | false | Client Reconnect | false, 4, {4, 60}

### Clean Session

Default Clean Session value is true, If you want to set Clean Session = false, add option <code>{clean_sess, false}</code>.

```erlang

emqttc:start_link([{host, "test.mosquitto.org"}, {clean_sess, false}]).

```

### KeepAlive

Default KeepAlive value is 60(secs), If you want to change KeepAlive, add option <code>{keepalive, 300}</code>. No KeepAlive to use <code>{keepalive, 0}</code>.

```erlang

emqttc:start_link([{host, "test.mosquitto.org"}, {keepalive, 60}]).

```

### SSL Socket

Connect to broker with SSL Socket:

```erlang

emqttc:start_link([{host, "test.mosquitto.org"}, {port, 8883}, ssl]).

```

### Logger

Use 'logger' option to configure emqttc log mechanism. Default log to stdout with 'info' level.

```erlang

%% log to stdout with info level
emqttc:start_link([{logger, info}]).

%% log to otp standard error_logger with warning level
emqttc:start_link([{logger, {otp, warning}}]).

%% log to lager with error level
emqttc:start_link([{logger, {lager, error}}]).

```

#### Logger modules

Module | Description
-------|------------
stdout | io:format
otp    | error_logger
lager  | lager

#### Logger Levels

```
all
debug
info
warning
error
critical
none
```

### Reconnect

Use 'reconect' option to configure emqttc reconnect policy. Default is 'false'.

Reconnect Policy: <code>{MinInterval, MaxInterval, MaxRetries}</code>.

```erlang

%% reconnect with 4(secs) min interval, 60 max interval  
emqttc:start_link([{reconnect, 4}]).

%% reconnect with 3(secs) min interval, 120(secs) max interval and 10 max retries.
emqttc:start_link([{reconnect, {3, 120, 10}}]).

```

## Subscribe and Publish

### Publish API

```erlang

%% publish(Client, Topic, Payload) with Qos0
emqttc:publish(Client, <<"/test/TopicA">>, <<"Payload...">>).

%% publish(Client, Topic, Payload, PubOpts) with options
emqttc:publish(Client, <<"/test/TopicA">>, <<"Payload...">>, [{qos, 1}, {retain true}]).

```

### Subscribe API

```erlang

%% subscribe topic with Qos0
emqttc:subscribe(Client, <<"Topic">>).

%% subscribe topic with qos
emqttc:subscribe(Client, {<<"Topic">>, 1}).
%% or
emqttc:subscribe(Client, <<"Topic">>, 1).

%% subscribe topics
emqttc:subscribe(Client, [{<<"Topic1">>, 1}, {<<"Topic2">>, 2}]).

%% unsubscribe
emqttc:unsubscribe(Client, <<"Topic">>).
emqttc:unsubscribe(Client, [<<"Topic1">>, <<"Topic2">>]).

```

## Ping and Pong

```erlang
pong = emqttc:ping(Client).
```

## Disconnect

```erlang
emqttc:disconnect(Client).
```

## Design 

![Design](https://raw.githubusercontent.com/emqtt/emqttc/master/doc/Socket.png)

## License

The MIT License (MIT)

## Contributors

@hiroeorz

@desoulter

## Contact

feng@emqtt.io
