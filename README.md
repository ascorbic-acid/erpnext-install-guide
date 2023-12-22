### Fast steps to install Frappe/ERPNext v14 on Clean Ubuntu 22.04 Production/Develop
##### feel free to edit the parameters to suite your needs
```
sudo apt-get update && sudo apt-get upgrade
```

```
sudo apt-get install software-properties-common \
wget zip unzip git curl certbot python3-pip \
python3-dev python3-venv redis-server mariadb-server \
mariadb-client xvfb libfontconfig xfonts-75dpi \
fontconfig libxrender1
```

#### get wkhtmltopdf qt patched
```
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-2/wkhtmltox_0.12.6.1-2.jammy_amd64.deb
sudo dpkg -i wkhtmltox_0.12.6.1-2.jammy_amd64.deb
```
#### config mariadb
```
sudo nano /etc/mysql/my.cnf
```

#### and add the following at the bottom then save
```
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
innodb_buffer_pool_size = 2048M # select your best value

[mysql]
default-character-set = utf8mb4
```


#### now lets restart mariadb
```
sudo service mariadb restart
```
```
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```
```
source ~/.bashrc; nvm install 18
```

#### add node & npm link to /usr/bin to avoid supervisor.conf not having node entry and therefore socketio io not working
```
sudo ln -s $(which node) /usr/bin/node
sudo ln -s $(which npm) /usr/bin/npm
```
```
npm install -g yarn
```

#### dont choose Y for: "Switch to unix_socket authentication"
```
sudo mysql_secure_installation
```
```
sudo pip3 install frappe-bench
```
#### init new frappe bench v14 env
```
bench init frappe-bench --version version-14 --verbose --install-app erpnext
```
```
cd frappe-bench && bench get-app payments --branch version-14; bench get-app hrms --branch version-14
```

### We are done now for the development phase, we can use bench start. To setup Production mode please continue the steps below

```
bench new-site my-site-name.local --install-app erpnext
```
```
sudo bench setup production $USER
sudo sed -i '6i chown='"$USER"':'"$USER"'' /etc/supervisor/supervisord.conf
```
```
bench config dns_multitenant on
sudo service supervisor restart
sudo bench setup production $USER
```
#### Enable scheduler
```
bench --site my-site-name.local scheduler enable
bench --site my-site-name.local scheduler resume
```
```
bench restart
```
#### fix permissions issue in Ubuntu 22.04, where page loads with missing styles
```
sudo chmod 755 /home/$(echo $USER)
```

# DONE
