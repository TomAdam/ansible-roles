GPG encryption key gen
======================

Don't generate keys within a VM. You will regret it.

1.  Create server key
    
        gpg2 --full-gen-key
    
    RSA and RSA, 2048 bit, no expiry.
    
2.  Create backup key

        gpg2 --full-gen-key
    
    RSA and RSA, 2048 bit, no expiry.
    
3.  Sign backup key with server key
        
        gpg2 -u <server_key_id> --edit-key <backup_key_id>
        sign

4.  Export owner trust db

        gpg2 --export-ownertrust
        
    Use the line that ends with the server key id
    
5.  Export keys

        gpg2 --armor --output server.key --export-secret-keys <server_key_id>
        gpg2 --armor --output server.pub --export <server_key_id>
        gpg2 --armor --output db_backup.key --export-secret-keys <backup_key_id>
        gpg2 --armor --output db_backup.pub --export <backup_key_id>

6.  Delete keys

        gpg2 --delete-secret-and-public-keys <server_key_id>
        gpg2 --delete-secret-and-public-keys <backup_key_id>
