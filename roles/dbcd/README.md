# Database Copy Down (DBCD)

## Server
Serves the database dump to the consumers. Read only access rsync access via the `dbcd_server_user` (default dbcd) for 
consumers and write only rsync access for the client.

### Required vars
- dbcd_mode = `["server"]`
- dbcd_server_client_public_key - The public key for write access, used by the client
- dbcd_server_consumer_public_keys - List of public keys for read access, used by the consumers

## Client
Sends the database dump to the server.

### Required vars
- dbcd_mode = `["client"]`
- dbcd_client_user - Username that will send dumps to the server
- dbcd_client_private_key - Private key for write access to the server
- dbcd_server_host
- dbcd_known_hosts - List of known hosts for the server, needed for automation

## Consumer
Logs in to the server to rsync the database dump.

### Required vars
- dbcd_mode = `["consumer"]`
- dbcd_client_user
- dbcd_consumer_user
- dbcd_server_host
