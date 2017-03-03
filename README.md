# chef-server

chef-server will run Chef Server 12 in an Ubuntu Trusty 14.04 LTS container.
Image Size: 1.124 GB

This is a fork of: [base/chef-server](https://registry.hub.docker.com/u/base/chef-server/).

## Environment
Chef is running over HTTPS/443 by default. You can however change that to another port by updating the `CHEF_PORT` variable and the expose port `-p`.

## Usage
*Launch the container:*

```
$ docker run --privileged -e CHEF_PORT=443 --name chef-server -d -p 443:443 cbuisson/chef-server
```

*Launch the container with logs volumes:*

```
$ docker run --privileged -e CHEF_PORT=443 --name chef-server -d -v ~/chef-logs:/var/log -v ~/install-chef-out:/root -p 443:443 cbuisson/chef-server
```

Once Chef Server 12 is configured, you can download the Knife admin keys here:

```
$ curl -Ok https://CONTAINER_ID:CHEF_PORT/knife_admin_key.tar.gz
```

Then un-tar that archive and point your config.rb to the `admin.pem` and `admin-validator.pem` files.

*config.rb* example:

```ruby
log_level                :info
log_location             STDOUT
cache_type               'BasicFile'
node_name                'admin'
client_key               '/home/cbuisson/.chef/admin.pem'
validation_client_name   'admin-validator'
validation_key           '/home/cbuisson/.chef/admin-validator.pem'
chef_server_url          'https://CONTAINER_ID:CHEF_PORT/organizations/my_org'
```
Note: CONTAINER_ID **needs** to be resolvable by hostname!

When the config.rb file is ready, you will need to get the SSL certificate files from the container to access Chef Server:

```bash
cbuisson@t530:~/.chef# knife ssl fetch
WARNING: Certificates from 512ab20b1e0d will be fetched and placed in your trusted_cert
directory (/home/cbuisson/.chef/trusted_certs).

Knife has no means to verify these are the correct certificates. You should
verify the authenticity of these certificates after downloading.

Adding certificate for 512ab20b1e0d in /home/cbuisson/.chef/trusted_certs/512ab20b1e0d.crt
```

You should now be able to use the knife command!
```bash
cbuisson@t530:~# knife user list
admin
```

##### Known issue
`chef-manage-ctl reconfigure` needs to run in order to access the Chef webui. When this command is executed within the container, it blocks here:
```bash
* ruby_block[wait for redis service socket] action run
```
Therefore the Chef Server 12 webui isn't available at the moment, however this isn't required to use Chef since Knife is working.

##### Tags
v1.0: Chef Server 11
v2.0: Chef Server 12
