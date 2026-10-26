Запустить приложение https://github.com/anestesia001/carss/tree/main как systemd демон

````
artsiom@ubundu26:/$ ll -la /opt/carss/
total 24
drwxr-xr-x  4 webadmin webgroup 4096 Oct 26 19:26 ./
drwxr-xr-x 12 root     root     4096 Oct 26 18:32 ../
drwxr-xr-x  2 webadmin webgroup 4096 Oct 26 19:26 __pycache__/
-rw-r--r--  1 webadmin webgroup 2814 Oct 26 18:33 main.py
-rw-r--r--  1 webadmin webgroup   42 Oct 26 18:33 requirements.txt
drwxr-xr-x  5 webadmin webgroup 4096 Oct 26 18:36 venv/
````
unit file
````
artsiom@ubundu26:/$ cat /etc/systemd/system/app-carss.service 
[Unit]
Description=app-carss
After=network.target

[Service]
Type=simple
User=webadmin
Group=webgroup
WorkingDirectory=/opt/carss
ExecStart=/opt/carss/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
EnvironmentFile=/etc/carss.env
Restart=on-failure
#Environment=DATABASE_URL=postgresql://postgres:mysecretpassword@127.0.0.1:5432/autos

[Install]
WantedBy=multi-user.target
````


````
artsiom@ubundu26:/$ sudo systemctl status app-carss.service 
● app-carss.service - app-carss
     Loaded: loaded (/etc/systemd/system/app-carss.service; disabled; preset: enabled)
     Active: active (running) since Mon 2026-10-26 19:45:14 +03; 46min ago
 Invocation: 2b7377cf33db48449a1258a22dd00cba
   Main PID: 504535 (uvicorn)
      Tasks: 6 (limit: 2338)
     Memory: 48.5M (peak: 48.8M)
        CPU: 11.200s
     CGroup: /system.slice/app-carss.service
             └─504535 /opt/carss/venv/bin/python3 /opt/carss/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
````
postgres DB
````
sudo -i -u postgres psql
CREATE USER carss_user WITH PASSWORD 'StrongPassw0rd!';
CREATE DATABASE autos OWNER carss_user;
GRANT ALL PRIVILEGES ON DATABASE autos TO carss_user;
\c autos
ALTER SCHEMA public OWNER TO carss_user;
GRANT ALL ON SCHEMA public TO carss_user;
\q
````

проверка
````

psql 'postgresql://carss_user:StrongPassw0rd!@localhost:5432/autos' -c "SELECT current_user, current_database();"
````
<img width="942" height="88" alt="Снимок экрана 2026-10-26 в 21 50 00" src="https://github.com/user-attachments/assets/a1ad02ba-8ee4-4e30-b58f-73ffacd1ab53" />


nginx конфиг
````
server {
    listen 80;
    server_name _;                                      

    access_log /var/log/nginx/carss.access.log;
    error_log  /var/log/nginx/carss.error.log;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_read_timeout 60s;
    }
}

````

активация nginx
````
sudo ln -sf /etc/nginx/sites-available/carss /etc/nginx/sites-enabled/carss
sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
sudo systemctl reload nginx
````
проверка

<img width="441" height="361" alt="Снимок экрана 2026-10-26 в 22 01 28" src="https://github.com/user-attachments/assets/0e00e0aa-7ad8-4967-a0f2-c1f6ad13a157" />



