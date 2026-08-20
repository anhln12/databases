```
Admin>SELECT variable_name, variable_value
    -> FROM global_variables
    -> WHERE variable_name LIKE '%mysql-%'
    -> ORDER BY variable_name;
+----------------------------------------------------------------------+--------------------------------------------+
| variable_name                                                        | variable_value                             |
+----------------------------------------------------------------------+--------------------------------------------+
| mysql-add_ldap_user_comment                                          |                                            |
| mysql-auditlog_filename                                              |                                            |
| mysql-auditlog_filesize                                              | 104857600                                  |
| mysql-auditlog_flush_size                                            | 4096                                       |
| mysql-auditlog_flush_timeout                                         | 1000                                       |
| mysql-aurora_max_lag_ms_only_read_from_replicas                      | 2                                          |
| mysql-auto_increment_delay_multiplex                                 | 5                                          |
| mysql-auto_increment_delay_multiplex_timeout_ms                      | 10000                                      |
| mysql-autocommit_false_is_transaction                                | false                                      |
| mysql-autocommit_false_not_reusable                                  | false                                      |
| mysql-automatic_detect_sqli                                          | false                                      |
| mysql-aws_blue_green_deployment_auto_discovery                       | 1                                          |
| mysql-binlog_reader_connect_retry_msec                               | 3000                                       |
| mysql-client_host_cache_size                                         | 0                                          |
| mysql-client_host_error_counts                                       | 0                                          |
| mysql-client_session_track_gtid                                      | true                                       |
| mysql-commands_stats                                                 | true                                       |
| mysql-connect_retries_delay                                          | 1                                          |
| mysql-connect_retries_on_failure                                     | 10                                         |
| mysql-connect_timeout_client                                         | 10000                                      |
| mysql-connect_timeout_server                                         | 3000                                       |
| mysql-connect_timeout_server_max                                     | 10000                                      |
| mysql-connection_delay_multiplex_ms                                  | 0                                          |
| mysql-connection_max_age_ms                                          | 0                                          |
| mysql-connection_warming                                             | false                                      |
| mysql-connpoll_reset_queue_length                                    | 50                                         |
| mysql-data_packets_history_size                                      | 0                                          |
| mysql-default_authentication_plugin                                  | mysql_native_password                      |
| mysql-default_charset                                                | utf8                                       |
| mysql-default_collation_connection                                   | utf8_general_ci                            |
| mysql-default_max_latency_ms                                         | 1000                                       |
| mysql-default_query_delay                                            | 0                                          |
| mysql-default_query_timeout                                          | 36000000                                   |
| mysql-default_reconnect                                              | true                                       |
| mysql-default_schema                                                 | information_schema                         |
| mysql-default_session_track_gtids                                    | OFF                                        |
| mysql-enable_client_deprecate_eof                                    | true                                       |
| mysql-enable_load_data_local_infile                                  | false                                      |
| mysql-enable_server_deprecate_eof                                    | true                                       |
| mysql-enforce_autocommit_on_reads                                    | false                                      |
| mysql-evaluate_replication_lag_on_servers_load                       | 1                                          |
| mysql-eventslog_buffer_history_size                                  | 0                                          |
| mysql-eventslog_buffer_max_query_length                              | 32768                                      |
| mysql-eventslog_default_log                                          | 0                                          |
| mysql-eventslog_filename                                             |                                            |
| mysql-eventslog_filesize                                             | 104857600                                  |
| mysql-eventslog_flush_size                                           | 4096                                       |
| mysql-eventslog_flush_timeout                                        | 1000                                       |
| mysql-eventslog_format                                               | 1                                          |
| mysql-eventslog_rate_limit                                           | 1                                          |
| mysql-eventslog_stmt_parameters                                      | 0                                          |
| mysql-eventslog_table_memory_size                                    | 10000                                      |
| mysql-fast_forward_grace_close_ms                                    | 5000                                       |
| mysql-firewall_whitelist_enabled                                     | false                                      |
| mysql-firewall_whitelist_errormsg                                    | Firewall blocked this query                |
| mysql-free_connections_pct                                           | 10                                         |
| mysql-handle_unknown_charset                                         | 1                                          |
| mysql-handle_warnings                                                | 1                                          |
| mysql-have_compress                                                  | true                                       |
| mysql-have_ssl                                                       | true                                       |
| mysql-hostgroup_manager_verbose                                      | 1                                          |
| mysql-ignore_min_gtid_annotations                                    | false                                      |
| mysql-init_connect                                                   |                                            |
| mysql-interfaces                                                     | 0.0.0.0:6033                               |
| mysql-keep_multiplexing_variables                                    | tx_isolation,transaction_isolation,version |
| mysql-kill_backend_connection_when_disconnect                        | true                                       |
| mysql-ldap_user_variable                                             |                                            |
| mysql-log_mysql_warnings_enabled                                     | false                                      |
| mysql-log_unhealthy_connections                                      | true                                       |
| mysql-long_query_time                                                | 1000                                       |
| mysql-max_allowed_packet                                             | 67108864                                   |
| mysql-max_connections                                                | 2048                                       |
| mysql-max_stmts_cache                                                | 10000                                      |
| mysql-max_stmts_per_connection                                       | 20                                         |
| mysql-max_transaction_idle_time                                      | 14400000                                   |
| mysql-max_transaction_time                                           | 14400000                                   |
| mysql-min_num_servers_lantency_awareness                             | 1000                                       |
| mysql-mirror_max_concurrency                                         | 16                                         |
| mysql-mirror_max_queue_length                                        | 32000                                      |
| mysql-monitor_aws_rds_topology_discovery_interval                    | 0                                          |
| mysql-monitor_connect_interval                                       | 2000                                       |
| mysql-monitor_connect_timeout                                        | 1000                                       |
| mysql-monitor_enabled                                                | true                                       |
| mysql-monitor_galera_healthcheck_interval                            | 5000                                       |
| mysql-monitor_galera_healthcheck_max_timeout_count                   | 3                                          |
| mysql-monitor_galera_healthcheck_timeout                             | 800                                        |
| mysql-monitor_groupreplication_healthcheck_interval                  | 5000                                       |
| mysql-monitor_groupreplication_healthcheck_max_timeout_count         | 3                                          |
| mysql-monitor_groupreplication_healthcheck_timeout                   | 800                                        |
| mysql-monitor_groupreplication_max_transactions_behind_count         | 3                                          |
| mysql-monitor_groupreplication_max_transactions_behind_for_read_only | 1                                          |
| mysql-monitor_history                                                | 600000                                     |
| mysql-monitor_local_dns_cache_refresh_interval                       | 60000                                      |
| mysql-monitor_local_dns_cache_ttl                                    | 300000                                     |
| mysql-monitor_local_dns_resolver_queue_maxsize                       | 128                                        |
| mysql-monitor_password                                               | iPpDUAm*50BPO%                             |
| mysql-monitor_ping_interval                                          | 2000                                       |
| mysql-monitor_ping_max_failures                                      | 3                                          |
| mysql-monitor_ping_timeout                                           | 500                                        |
| mysql-monitor_query_interval                                         | 60000                                      |
| mysql-monitor_query_timeout                                          | 100                                        |
| mysql-monitor_read_only_interval                                     | 2000                                       |
| mysql-monitor_read_only_max_timeout_count                            | 3                                          |
| mysql-monitor_read_only_timeout                                      | 500                                        |
| mysql-monitor_replication_lag_count                                  | 1                                          |
| mysql-monitor_replication_lag_group_by_host                          | false                                      |
| mysql-monitor_replication_lag_interval                               | 10000                                      |
| mysql-monitor_replication_lag_timeout                                | 1000                                       |
| mysql-monitor_replication_lag_use_percona_heartbeat                  |                                            |
| mysql-monitor_slave_lag_when_null                                    | 60                                         |
| mysql-monitor_threads_max                                            | 128                                        |
| mysql-monitor_threads_min                                            | 8                                          |
| mysql-monitor_threads_queue_maxsize                                  | 128                                        |
| mysql-monitor_username                                               | monitor                                    |
| mysql-monitor_wait_timeout                                           | true                                       |
| mysql-monitor_writer_is_also_reader                                  | true                                       |
| mysql-multiplexing                                                   | true                                       |
| mysql-parse_failure_logs_digest                                      | false                                      |
| mysql-passthrough_auth_cache_ttl_s                                   | 0                                          |
| mysql-passthrough_auth_empty_password                                | true                                       |
| mysql-passthrough_auth_enabled                                       | false                                      |
| mysql-passthrough_auth_failure_map_cap                               | 100000                                     |
| mysql-passthrough_auth_failure_window_s                              | 60                                         |
| mysql-passthrough_auth_max_failures_per_ip                           | 10                                         |
| mysql-passthrough_auth_max_failures_per_user                         | 3                                          |
| mysql-passthrough_auth_max_inflight_probes                           | 100                                        |
| mysql-passthrough_auth_require_tls                                   | true                                       |
| mysql-passthrough_auth_unknown_users                                 | false                                      |
| mysql-passthrough_auth_username_pattern                              |                                            |
| mysql-passthrough_default_hg                                         | 0                                          |
| mysql-passthrough_default_schema                                     |                                            |
| mysql-ping_interval_server_msec                                      | 120000                                     |
| mysql-ping_timeout_server                                            | 500                                        |
| mysql-poll_timeout                                                   | 2000                                       |
| mysql-poll_timeout_on_failure                                        | 100                                        |
| mysql-processlist_max_query_length                                   | 2097152                                    |
| mysql-protocol_compression_level                                     | 3                                          |
| mysql-proxy_protocol_networks                                        |                                            |
| mysql-query_cache_handle_warnings                                    | 0                                          |
| mysql-query_cache_size_MB                                            | 256                                        |
| mysql-query_cache_soft_ttl_pct                                       | 0                                          |
| mysql-query_cache_stores_empty_result                                | true                                       |
| mysql-query_digests                                                  | true                                       |
| mysql-query_digests_grouping_limit                                   | 3                                          |
| mysql-query_digests_groups_grouping_limit                            | 10                                         |
| mysql-query_digests_keep_comment                                     | false                                      |
| mysql-query_digests_lowercase                                        | false                                      |
| mysql-query_digests_max_digest_length                                | 2048                                       |
| mysql-query_digests_max_query_length                                 | 65000                                      |
| mysql-query_digests_no_digits                                        | false                                      |
| mysql-query_digests_normalize_digest_text                            | false                                      |
| mysql-query_digests_replace_null                                     | false                                      |
| mysql-query_digests_track_hostname                                   | false                                      |
| mysql-query_processor_first_comment_parsing                          | 2                                          |
| mysql-query_processor_iterations                                     | 0                                          |
| mysql-query_processor_parser                                         | 0                                          |
| mysql-query_processor_regex                                          | 1                                          |
| mysql-query_retries_on_failure                                       | 1                                          |
| mysql-query_rules_fast_routing_algorithm                             | 1                                          |
| mysql-reset_connection_algorithm                                     | 2                                          |
| mysql-resolution_family                                              | system                                     |
| mysql-select_version_forwarding                                      | 3                                          |
| mysql-server_capabilities                                            | 569867                                     |
| mysql-server_version                                                 | 5.5.30                                     |
| mysql-servers_stats                                                  | true                                       |
| mysql-session_idle_ms                                                | 1                                          |
| mysql-session_idle_show_processlist                                  | true                                       |
| mysql-session_track_variables                                        | 0                                          |
| mysql-sessions_sort                                                  | true                                       |
| mysql-set_parser_algorithm                                           | 2                                          |
| mysql-set_query_lock_on_hostgroup                                    | 1                                          |
| mysql-show_processlist_extended                                      | 0                                          |
| mysql-shun_on_failures                                               | 5                                          |
| mysql-shun_recovery_time_sec                                         | 10                                         |
| mysql-ssl_p2s_ca                                                     |                                            |
| mysql-ssl_p2s_capath                                                 |                                            |
| mysql-ssl_p2s_cert                                                   |                                            |
| mysql-ssl_p2s_cipher                                                 |                                            |
| mysql-ssl_p2s_crl                                                    |                                            |
| mysql-ssl_p2s_crlpath                                                |                                            |
| mysql-ssl_p2s_key                                                    |                                            |
| mysql-stacksize                                                      | 1048576                                    |
| mysql-stats_time_backend_query                                       | false                                      |
| mysql-stats_time_query_processor                                     | false                                      |
| mysql-tcp_keepalive_time                                             | 120                                        |
| mysql-threads                                                        | 4                                          |
| mysql-threshold_query_length                                         | 524288                                     |
| mysql-threshold_resultset_size                                       | 4194304                                    |
| mysql-throttle_connections_per_sec_to_hostgroup                      | 1000000                                    |
| mysql-throttle_max_bytes_per_second_to_client                        | 0                                          |
| mysql-throttle_ratio_server_to_client                                | 0                                          |
| mysql-unshun_algorithm                                               | 0                                          |
| mysql-use_tcp_keepalive                                              | true                                       |
| mysql-verbose_query_error                                            | false                                      |
| mysql-wait_timeout                                                   | 28800000                                   |
| mysql-zstd_compression_level                                         | 3                                          |
+----------------------------------------------------------------------+--------------------------------------------+
196 rows in set (0.01 sec)
```
