# ThingsboardProject

Geliştirme ortamı olarak Oracle VM VirtualBox kullanıldı.  
Yaratılan sanal makina üzerine Ubuntu 64bit 18.04 server kurulumu yapıldı.  
Sanal makina üzerine Java 11 kurulumu yapıldı:  
```
sudo apt update
sudo apt install openjdk-11-jdk
```
Sanal makina üzerine PostgreSQL 12 kurulumu yapıldı:  
```
# install **wget** if not already installed:
sudo apt install -y wget

# import the repository signing key:
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -

# add repository contents to your system:
RELEASE=$(lsb_release -cs)
echo "deb http://apt.postgresql.org/pub/repos/apt/ ${RELEASE}"-pgdg main | sudo tee  /etc/apt/sources.list.d/pgdg.list

# install and launch the postgresql service:
sudo apt update
sudo apt -y install postgresql-12
sudo service postgresql start
```

PostgreSQL için kullanıcı ve şifre ayarı yapıldı:  
```
sudo su - postgres
psql
\password
\q
```
Thingsboard veritabanı yaratıldı:  
```
psql -U postgres -d postgres -h 127.0.0.1 -W
CREATE DATABASE thingsboard;
\q
```
Maven kurulumu yapıldı:   
```
sudo apt-get install maven
```
Thingsboard kütüphanesi klonlandı:   
```
git clone https://github.com/thingsboard/thingsboard.git
```
İndirilen kütüphane derlenerek build alındı:  
```
mvn clean install -DskipTests
```
Thingsboard conf dosyasından PostgreSQL veritabanı bağlantısı sunucuya yönlendirildi ve konfigüre edildi:  
```
sudo nano /etc/thingsboard/conf/thingsboard.conf
```
```
# DB Configuration 
export DATABASE_ENTITIES_TYPE=sql
export DATABASE_TS_TYPE=sql
export SPRING_JPA_DATABASE_PLATFORM=org.hibernate.dialect.PostgreSQLDialect
export SPRING_DRIVER_CLASS_NAME=org.postgresql.Driver
export SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/thingsboard
export SPRING_DATASOURCE_USERNAME=postgres
export SPRING_DATASOURCE_PASSWORD=PUT_YOUR_POSTGRESQL_PASSWORD_HERE
export SPRING_DATASOURCE_MAXIMUM_POOL_SIZE=5
# Specify partitioning size for timestamp key-value storage. Allowed values: DAYS, MONTHS, YEARS, INDEFINITE.
export SQL_POSTGRES_TS_KV_PARTITIONING=MONTHS
```

Alınan build neticesinde Ubuntu işletim sistemi için oluşan thingsboard.deb dosyası kuruldu:  
```
sudo dpkg -i thingsboard.deb
```
thingsboard install.sh scripti çalıştırılarak demo database ve schema yaratılmış oldu:    
```
sudo /usr/share/thingsboard/bin/install/install.sh --loadDemo
```

thingsboard servisi başlatıldı:  
```
sudo service thingsboard start
```
Varsayılan hesap şifreleri değiştirildi. (sysadmin, tenant, customer)  
Yeni birer tenant, customer ve device oluşturuldu.  

Sıcaklık ve nem verilerinin takibini sağlamak için dashboard yaratıldı.  

Yaratılan customer hesabının ana ekranı dashboard sayfasına yönlendirildi.  

Python ile paho-mqtt kütüphanesi kullanılarak sunucuya beş saniyede bir JSON formatında sıcaklık ve nem verileri gönderecek bir uygulama geliştirildi. (Uygulama 30-35 °C aralığında sıcaklık verisini ve 60-80 (%) aralığında nem verisini rastgele üreterek beş saniyede bir servise göndermektedir.) 

Sunucuya taşımadan önce logo ve favicon görselleri firma logosu ve ikonlarıyla değiştirilerek yeniden build alındı ve deb paketi elde edildi.  

DigitalOcean'dan yeni bir Ubuntu sunucu yaratılarak geliştirme ortamına benzer bir sunucu kurulumu yapılarak geliştirme ortamında oluşturulan deb dosyası (logo ve favicon görselleri özelleştirilmiş .deb paketi) DigitalOcean üzerindeki sunucuya kuruldu.  

Geliştirme ortamındaki veritabanını sunucuya taşımak için PostgreSQL'in backup ve restore tool'ları (pg_dump, pqsl)  kullanıldı.  

DigitalOcean üzerindeki aynı sunucu üzerinde python kullanılarak hazırlanan aygıt simülator yazılımı Linux'un nohup komutuyla arka plana alındı:  
```
nohup ./device.py &
```
