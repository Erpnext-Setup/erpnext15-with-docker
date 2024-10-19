# ErpNext15 with docker
![Alt text](https://github.com/Erpnext-Setup/erpnext15-with-docker/blob/main/Youtube%20Thumbnail.png)

##  pull image
```
docker pull 1devops2/erpnextdb:15 				
docker pull 1devops2/erpnext:15
```

## create docker network
docker network for connectivity between erp and database container. otherwise they can't communicate with each others.
```
docker network create erpnet
```

## create `conf` directory & copy mariadb conf
this conf is required for our mariadb container and we need to mount into the container.
```
mkdir conf; cd conf
wget https://raw.githubusercontent.com/Erpnext-Setup/erpnext14-with-docker/main/conf/mariadb.cnf
```

## run mariadb container
lets run the maraidb container, always make sure you are mounting from the correct path.
```
docker run -it  --net erpnet --name erpdb -e MYSQL_ROOT_PASSWORD=1QWERT2 -v /root/conf:/etc/mysql/conf.d -d 1devops2/erpnextdb:15
```
⚠ note: change conf path, i-e `/root/conf`


## run erp container
```
docker run -it --network erpnet --name erp -p 8000:8000 -p 9000:9000 -p 3306:3306 -d 1devops2/erpnext:15
```

## exec into erp container
now containers are running. lets install our erp site. `1devops2.blog` is my FQDN domain. you need to update it. if you are doing locally testing then let it rest.
```
docker exec -it erp bash
cd frappe-bench && bench new-site 1devops2.blog --admin-password '321' --mariadb-root-username root --db-host erpdb  # db-pass: 1QWERT2
bench --site 1devops2.blog install-app erpnext
```

### install pos-awesome [optional]
```
bench get-app branch version-14 https://github.com/yrestom/POS-Awesome.git
bench setup requirements
bench build --app posawesome
bench --site 1devops2.blog install-app posawesome
```

### start erp
```
bench use 1devops2.blog
cd /home/frappe/frappe-bench && bench start
```
## For Local Setup: set hostname & add entry into `/etc/hosts`
### set hostname
we need to set the hostname
```
hostnamectl set-hostname 1devops2.blog
exec bash  # this command will update the hostname without restarting the machine
```
### add entry in /etc/hosts
This step is required, if you are doing testing locally
```
nano /etc/hosts
127.0.0.1 1devops2.blog 1devops2
```

### access erp     
```
http://1devops2.blog:8000
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
    server_name 1devops2.blog www.1devops2.blog;

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
http://1devops2.blog/app
```

## Secure Nginx with Let's Encrypt
If you're testing locally, such as on VMware Workstation, you can skip that part.
### Install Certbot
```
sudo apt install certbot python3-certbot-nginx -y
```

### obtain SSL certificates
```
sudo certbot --nginx -d 1devops2.blog -d www.1devops2.blog
```
## Ports
```
80 443 8000 9000 3306
```
