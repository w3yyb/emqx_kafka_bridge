
# emqx_kafka_bridge(emqx V3.1.* kafka插件)

This is a plugin for the EMQX broker that sends all messages received by the broker to kafka.注意：需要首先安装 erlang/opt 21.x版本。
## V2.x版的插件请参考 https://github.com/w3yyb/emq_kafka_bridge

## Build the EMQX broker

1. Clone emqx-relx project

   We need to clone the EMQX project [GITHUB](https://github.com/emqx/emqx-rel)

```shell
  wget https://github.com/emqx/emqx-rel/archive/v3.1.1.zip
  unzip v3.1.1.zip
```

2. Add EMQ Kafka bridge as a DEPS
   Adding EMQ kafka bridge as a dependency in the Makefile.

   1. search for `DEPS += $(foreach dep,$(MAIN_APPS),$(call app_name,$(dep)))`
      add the following line before the above lines
      > DEPS += emqx_kafka_bridge

   2. search for
     ```makefile
     $(foreach dep,$(MAIN_APPS),$(eval dep_$(call app_name,$(dep)) = $(CLONE_METHOD) https://github.com/emqx/$(dep) $(call app_vsn,$(dep))))
     ```
     add the following line before the above lines
     >dep_emqx_kafka_bridge = git https://github.com/w3yyb/emqx_kafka_bridge.git master

3. Add load plugin in relx.config
   >{emqx_kafka_bridge, load},

4. Build
   ```shell
   cd emqx-relx && make
   ```

Configuration
----------------------
You will have to edit the configurations of the bridge to set the kafka `Ip address and port`.
and  `afka.payloadtopic = Payload`

Edit the file `emq-relx/deps/emqx_kafka_bridge/etc/emqx_kafka_bridge.conf`

```conf
##--------------------------------------------------------------------
## kafka Bridge
##--------------------------------------------------------------------

## The Kafka loadbalancer node host that bridge is listening on.
##
## Value: 127.0.0.1, localhost
kafka.host = 127.0.0.1

## The kafka loadbalancer node port that bridge is listening on.
##
## Value: Port
kafka.port = 9092

## The kafka loadbalancer node partition strategy.
##
## Value: strict_round_robin
kafka.partitionstrategy = strict_round_robin

## payload topic.
##
## Value: string
kafka.payloadtopic = Payload


```
edit    `rel/conf/plugins/emqx_kafka_bridge.conf`  and `_rel/emqx/etc/plugins/emqx_kafka_bridge.conf`
`IP and port ` and `kafka.payloadtopic = Payload`

Start the EMQ broker and load the plugin 
-----------------
1) cd emqx-relx/_rel/emqx
2) ./bin/emqx start
3) ./bin/emqx_ctl plugins load emqx_kafka_bridge

Test
-----------------
Send a MQTT message on a random topic from a MQTT client to your EMQ broker.

The following should be received by your kafka consumer :

  {"topic":"yourtopic", "message":[yourmessage]}
This is the format in which kafka will receive the MQTT messages

If Kafka consumer shows no messages even after publishing to EMQX - ACL makes the plugin fail, so please remove all the ACL related code to ensure it runs properly. We will soon push the updated (Working) code to the repository. 

## License

This project is licensed under the Apache 2.0 License - see the [LICENSE](LICENSE) file for details

