# erpnext15 with docker

##  pull image
```
docker pull 1devops2/erpnextdb:15 				
docker pull 1devops2/erpnext:15
```

## create docker network
```
docker network create erpnet
```

## create `conf` directory & copy mariadb conf
```
mkdir conf; cd conf
wget https://raw.githubusercontent.com/Erpnext-Setup/erpnext14-with-docker/main/conf/mariadb.cnf
```

## run mariadb container
```
docker run -it  --net erpnet --name erpdb -e MYSQL_ROOT_PASSWORD=1QWERT2 -v /root/conf:/etc/mysql/conf.d -d 1devops2/erpnextdb:15
```
⚠ note: change conf path, i-e `/root/conf`


## run erp container
```
docker run -it --network erpnet --name erp -p 8000:8000 -p 9000:9000 -p 3306:3306 -d 1devops2/erpnext:15
```

## exec into erp container
```
docker exec -it erp bash
cd frappe-bench && bench new-site erp.local --admin-password '321' --mariadb-root-username root --db-host erpdb  # db-pass: 1QWERT2
bench --site erp.local install-app erpnext
```

### install pos-awesome [optional]
```
bench get-app branch version-14 https://github.com/yrestom/POS-Awesome.git
bench setup requirements
bench build --app posawesome
bench --site erp.local install-app posawesome
```

### start erp
```
bench use erp.local 
cd /home/frappe/frappe-bench && bench start
```
## For Local Setup: set hostname & add entry into `/etc/hosts`
### set hostname
```
hostnamectl set-hostname erp.local
exec bash
```
### add entry in /etc/hosts
```
nano /etc/hosts
127.0.0.1 erp.local erp
```

### access erp     
```
http://erp.local:8000
```

## setup nginx as reverse proxy
### install nginx
```
apt-get install nginx -y
systemctl status nginx
```
### conf for erp
```
sudo cat << 'EOF' >> /etc/nginx/sites-available/erp
server {
    listen 80;
    server_name erp.local;

    location / {
        proxy_pass http://localhost:8000;  
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    }
}
EOF
```
### enable conf and restart nginx
```
ln -s /etc/nginx/sites-available/erp /etc/nginx/sites-enabled/erp
rm /etc/nginx/sites-enabled/default
systemctl reload nginx
```

### access with nginx
```
http://erp.local/app
```
