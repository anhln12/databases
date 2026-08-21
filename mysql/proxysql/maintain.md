A. Users
```
SELECT
    username,
    active,
    default_hostgroup,
    transaction_persistent,
    fast_forward,
    multiplex
FROM mysql_users;
```

B. Query rules
```
SELECT
    rule_id,
    active,
    username,
    schemaname,
    flagIN,
    digest,
    match_digest,
    match_pattern,
    destination_hostgroup,
    apply
FROM mysql_query_rules
ORDER BY rule_id;
```

C. Servers
```
SELECT
    hostgroup_id,
    hostname,
    port,
    status,
    weight,
    max_connections,
    max_replication_lag,
    use_ssl,
    comment
FROM mysql_servers
ORDER BY hostgroup_id, hostname;
```

D. Rumtime hostgroup
```
SELECT
    hostgroup_id,
    hostname,
    port,
    status,
    weight,
    max_connections,
    max_replication_lag,
    use_ssl,
    comment
FROM mysql_servers
ORDER BY hostgroup_id, hostname;
```
