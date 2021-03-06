## Custom Installation Guide (pfSense/OPNsense) + Elastic Stack 

## Table of Contents

- [Preparation](#preparation)
- [Installation](#installation)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)

# Preparation

### 1. Add Prerequisites
```
apt-get install apt-transport-https gnupg2 software-properties-common dirmngr lsb-release ca-certificates
```

### 2. Download MaxMind
```
wget https://github.com/maxmind/geoipupdate/releases/download/v4.3.0/geoipupdate_4.3.0_linux_amd64.deb
```

### 3. Install MaxMind
```
apt install ./geoipupdate_4.3.0_linux_amd64.deb
```

### 4. Download and install the public GPG signing key
```
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
```

### 5. Add Elasticsearch|Logstash|Kibana Repositories (version 7+)
```
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee -a /etc/apt/sources.list.d/elastic-7.x.list
```

### 6. Update
```
apt-get update
```

### 7. Install Java 14 LTS
```
apt install openjdk-14-jre-headless
```

### 8. Configure MaxMind
- Create a MaxMind Account @ https://www.maxmind.com/en/geolite2/signup
- Login to your MaxMind Account; navigate to "My License Key" under "Services" and Generate new license key
```
nano /etc/GeoIP.conf
```
- Modify lines 7 & 8 as follows (without < >):
```
AccountID <Input Your Account ID>
LicenseKey <Input Your LicenseKey>
```
- Modify line 13 as follows:
```
EditionIDs GeoLite2-City GeoLite2-Country GeoLite2-ASN
```
- Modify line 18 as follows:
```
DatabaseDirectory /usr/share/GeoIP/
```

### 9. Download MaxMind Databases
```
sudo geoipupdate 
```

### 10. Add cron (automatically updates MaxMind everyweek on Sunday at 1700hrs)
```
sudo nano /etc/cron.weekly/geoipupdate
```
- Add the following and save/exit
```
00 17 * * 0 geoipupdate
```

# Installation
- Elasticsearch v7+ | Kibana v7+ | Logstash v7+

### 11. Install Elasticsearch|Kibana|Logstash
```
apt-get install elasticsearch kibana logstash
```

# Configuration

### 12. Configure Kibana
```
sudo nano /etc/kibana/kibana.yml
```

### 13. Modify host file (/etc/kibana/kibana.yml)
- server.port: 5601
- server.host: "0.0.0.0"

### 14. Change Directory
```
cd /etc/logstash/conf.d/
```

### 15. (Required) Download the following configuration files
```
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/01-inputs.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/02-types.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/03-filter.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/05-firewall.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/30-geoip.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/50-outputs.conf
```

### 15a. (Optional) Download the following configuration files
```
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/10-others.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/35-rules-desc.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/36-ports-desc.conf
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/45-cleanup.conf
```

### 16a. Make Patterns Folder
```
sudo mkdir /etc/logstash/conf.d/patterns
```

### 16b. Navigate to Patterns Folder
```
cd /etc/logstash/conf.d/patterns
```

### 16c. Download the grok pattern
```
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/patterns/pfelk.grok
```

### 17a. Make Templates Folder
```
sudo mkdir /etc/logstash/conf.d/templates
```

### 17b. Navigate to Templates Folder
```
cd /etc/logstash/conf.d/templates
```

### 17c. Download Template(s)
```
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/templates/pfelk-geoip.json
```

### 18a. Make Databases Folder
```
sudo mkdir /etc/logstash/conf.d/databases
```

### 18a. Navigate to Databases Folder
```
cd /etc/logstash/conf.d/databases
```

### 18b. Download the Database(s)
```
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/databases/rule-names.csv
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/etc/logstash/conf.d/databases/service-names-port-numbers.csv
```

### 19. Enter your pfSense/OPNsense IP address (/data/pfELK/configurations/02-types.conf)
```
Change line 5; the "if [host] == ..." should point to your pfSense/OPNsense IP address
Change line 8; rename "firewall" (OPTIONAL) to identify your device (i.e. backup_firewall)
Change line 11-20; (OPTIONAL) to point to your second PF IP address or ignore
```

# Troubleshooting
### 20. Create Logging Directory 
```
mkdir -p /etc/pfELK/logs
```

### 21. Navigate to pfELK 
```
cd /data/pfELK/
```

### 22. Download `error-data.sh`
```
sudo wget https://raw.githubusercontent.com/3ilson/pfelk/master/error-data.sh
```

### 23. Make `error-data.sh` Executable
```
sudo chmod +x /data/pfELK/error-data.sh
```

### 24. Complete Configuration --> [Configuration](configuration.md)
