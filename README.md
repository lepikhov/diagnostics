# Диагностика для шкафов МАЛС

## 1. Базовые установки

Предполагается, что на диагностическом компьютере установлен ***AstraLinux***, создан пользователь ***tehnoprog***, установлен ***git*** и сделаны сетевые настройки в соответствии с проектом для шкафа МАЛС. На компьютер осуществлён вход под ***tehnoprog***

```bash
sudo apt update
sudo apt install mc nano nginx chromium
cd /home/tehnoprog
mkdir projects
cd projects
git clone github.com/lepikhov/diagnostics.git
cd diagnostics
git clone github.com/lepikhov/diagnostics-client.git
git clone github.com/lepikhov/diagnostics-service.git
```

## 2. Настройки параметров для каждой диагностируемой метрики 
```bash
nano /home/tehnoprog/projects/diagnostics/config.json
```

## 3. Установка и настройка mongodb
### 3.1. Установка 
```bash
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-debian10-5.0.16.tgz
sudo tar -zxvf mongodb-linux-x86_64-debian10-5.0.16.tgz
sudo mkdir -p /mongodb
sudo cp -R -n mongodb-linux-x86_64-debian10-5.0.16/bin/* /mongodb
sudo mkdir -p /db/mongodb_data/
export PATH=/mongodb:$PATH
sudo useradd --system --no-create-home mongod
sudo addgroup --system mongod
sudo chown -R mongod:mongod /mongodb/ /db/mongodb_data/
sudo cp /home/tehnoprog/projects/diagnostics/scripts/mongod.service /etc/systemd/system/
sudo touch /etc/mongod.conf
sudo mkdir /var/log/mongodb/
sudo chown -R mongod:mongod /mongodb/ /etc/mongod.conf /db/mongodb_data/ /var/log/mongodb/
```
### 3.2. Запуск сервиса
```bash
sudo systemctl daemon-reload
sudo systemctl enable mongod.service
sudo systemctl start mongod
sudo systemctl status mongod 
```

## 4. Настроки diagnostics-client
### 4.1. Найти в интернете и скопировать шрифты и иконки material design
- **/home/tehnoprog/projects/diagnostics/diagnostics-client/css/material-design/material-design-icons-4.0.0/font/MaterialIcons-Regular.woff, ...woff2**

- **/home/tehnoprog/projects/diagnostics/diagnostics-client/css/material-design/Roboto/Roboto-Regular.woff, ...woff2**

### 4.2. Протестировать запуск клиента
```bash
chromium --kiosk /home/tehnoprog/projects/diagnostics/diagnostics-client/index.html
```
### 4.3. Запуск сервиса
```bash
sudo cp /home/tehnoprog/projects/diagnostics/scripts/diagnostics-client.service /etc/systemd/user/
systemctl --user start diagnostics-client.service
systemctl --user enable diagnostics-client.service
systemctl --user status diagnostics-client.service
```

## 5. Настроки diagnostics-service
```bash
cd ./diagnostics-service
```
### 5.1. Установка python
```bash
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.14
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.14 1
python3.14 --version
sudo update-alternatives --config python3
```
### 5.2. Создание виртуального окружения и установка зависимостей
```bash
python3.14 -m venv venv
source ./venv/bin/activate
python -m pip install --upgrade pip
pip install flask flask_cors gunicorn pymongo json5 minimalmodbus pysoem
```
### 5.3. Настройка последовательных портов, Ethernet для EtherCAT и т.д. для данного компьютера
```bash
nano ./app/settings.py 
```
### 5.4. Проверка запуска под development server и gunicorn
```bash
python ./diagnostics_service.py
gunicorn --bind 0.0.0.0:5000 wsgi:app
```
```bash
deactivate
```
### 5.5. Запуск сервиса под gunicorn
```bash
sudo cp /home/tehnoprog/projects/diagnostics/scripts/diagnostics-service.service /etc/systemd/system/
sudo mkdir /gunicorn
sudo chown tehnoprog:www-data /gunicorn
sudo systemctl start diagnostics-service
sudo systemctl status diagnostics-service
sudo systemctl enable diagnostics-service
```
### 5.6. Настройка nginx и брандмауэра
#### Настройка ip сервера в конфигурационном файле nginx
```bash
nano /home/tehnoprog/projects/diagnostics/scripts/dianostics-service 
```

```bash
sudo cp /home/tehnoprog/projects/diagnostics/scripts/dianostics-service /etc/nginx/sites-available/
sudo ln -s /etc/nginx/sites-available/diagnostics-service /etc/nginx/sites-enabled
sudo nginx -t
sudo nginx -s reload
sudo systemctl restart nginx
sudo ufw delete allow 5000
sudo ufw allow 'Nginx Full'
```







