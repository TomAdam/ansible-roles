# Settings

### `mysql_innodb_buffer_pool_size`
This value must be set. Max 0.5 * RAM on a server where MySQL is shared with other services. 

### `mysql_innodb_flush_log_at_trx_commit`
This value defaults to 1. In a VM, set to 0 for better performance but less reliability.

### `mysql_innodb_log_files_in_group`
This value defaults to 2. Set to 4 if >2GB RAM.

### `mysql_innodb_log_file_size`
This value defaults to 128M. Set to 256M if >8GB RAM.

### `mysql_slow_query_log`
This value defaults to 0. Set to 1 to enable slow query logging.
