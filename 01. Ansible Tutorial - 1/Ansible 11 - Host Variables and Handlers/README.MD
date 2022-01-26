## 11. Host Variables and Handlers

```
mkdir host_vars

cd host_vars

cat <<EOF > 192.168.219.101.yml
apache_package_name: apache2
apache_service: apache2
php_package_name: libapache2-mod-php
EOF

cp 192.168.219.101.yml 192.168.219.102.yml

cat <<EOF > 192.168.219.103.yml
apache_package_name: httpd
apache_service: httpd
php_package_name: php
EOF
```