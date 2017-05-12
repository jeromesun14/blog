title: p4 simple_switch_CLI 使用说明
date: 2017-05-12 11:00:00
toc: true
tags: [p4]
categories: p4
keywords: [p4, p4app, simple_switch, bmv2, CLI, simple_switch_CLI, bm_CLI]
description: p4 simple_switch_CLI 使用说明
---

## help

在 simple_switch_CLI 提示符下，执行 `help`，即可显示帮助。以 p4app 下的 simple_switch_CLI 为例。

```
sunyongfeng@openswitch-OptiPlex-380:~/workshop/p4app/examples/bras.p4app$ docker exec -t -i a070609fbb24 simple_switch_CLI
Obtaining JSON from switch...
Done
Control utility for runtime P4 table manipulation
RuntimeCmd: help  

Documented commands (type help <topic>):
========================================
act_prof_add_member_to_group       set_crc16_parameters                   
act_prof_create_group              set_crc32_parameters                   
act_prof_create_member             set_queue_depth                        
act_prof_delete_group              set_queue_rate                         
act_prof_delete_member             shell                                  
act_prof_dump                      show_actions                           
act_prof_dump_group                show_ports                             
act_prof_dump_member               show_tables                            
act_prof_modify_member             swap_configs                           
act_prof_remove_member_from_group  switch_info                            
counter_read                       table_add                              
counter_reset                      table_delete                           
get_time_elapsed                   table_dump                             
get_time_since_epoch               table_dump_entry                       
help                               table_dump_entry_from_key              
load_new_config_file               table_dump_group                       
mc_dump                            table_dump_member                      
mc_mgrp_create                     table_indirect_add                     
mc_mgrp_destroy                    table_indirect_add_member_to_group     
mc_node_associate                  table_indirect_add_with_group          
mc_node_create                     table_indirect_create_group            
mc_node_destroy                    table_indirect_create_member           
mc_node_dissociate                 table_indirect_delete                  
mc_node_update                     table_indirect_delete_group            
mc_set_lag_membership              table_indirect_delete_member           
meter_array_set_rates              table_indirect_modify_member           
meter_get_rates                    table_indirect_remove_member_from_group
meter_set_rates                    table_indirect_set_default             
mirroring_add                      table_indirect_set_default_with_group  
mirroring_delete                   table_info                             
port_add                           table_modify                           
port_remove                        table_num_entries                      
register_read                      table_set_default                      
register_reset                     table_set_timeout                      
register_write                     table_show_actions                     
reset_state                        write_config_to_file                   
serialize_state                  

Undocumented commands:
======================
EOF  greet

RuntimeCmd: 
```

如果想查看每个命令的用法，以添加表项为例，执行 `help table_add`。

```
RuntimeCmd: help table_add
Add entry to a match table: table_add <table name> <action name> <match fields> => <action parameters> [priority]
RuntimeCmd: 
```

## 表操作

* 查看本 switch 所有表，`show_tables`
* 查看某张表的描述，`table_info 表名`
* 查看某张表的信息，`table_show 表名`
* 添加表项，`table_add 表名 action名 key值 => 字段1值 字段2值 字段3值`


