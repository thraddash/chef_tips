# chef_tips

https://github.com/chef/chef-server/issues/544

# create pivotal's public key from /etc/opscode/pivotal.pem and store in an accessible location

openssl rsa -in /etc/opscode/pivotal.pem -pubout > /var/opt/opscode/postgresql/9.2/data/pivotal.pub

# get the pivotal user's authz_id and store in an accessible location

echo "SELECT authz_id FROM auth_actor WHERE id = 1" | su -l opscode-pgsql -c 'psql bifrost -tA' | tr -d '\n' > /var/opt/opscode/postgresql/9.2/data/pivotal.authz_id

# create the pivotal user's record 

echo "INSERT INTO users (id, authz_id, username, email, pubkey_version, public_key, serialized_object, last_updated_by, created_at, updated_at) VALUES (md5(random()::text), pg_read_file('pivotal.authz_id'), 'pivotal', 'kryptonite@opscode.com', 0, pg_read_file('pivotal.pub'), '{\"first_name\":\"Clark\",\"last_name\":\"Kent\",\"display_name\":\"Clark Kent\"}', pg_read_file('pivotal.authz_id'), LOCALTIMESTAMP, LOCALTIMESTAMP);" | su -l opscode-pgsql -c 'psql opscode_chef'

# delete the temporary files

rm /var/opt/opscode/postgresql/9.2/data/pivotal.pub /var/opt/opscode/postgresql/9.2/data/pivotal.authz_id
