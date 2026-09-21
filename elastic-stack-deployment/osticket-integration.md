# osTicket Ticketing System Integration

## What is osTicket and why does a SOC use it?

osTicket is an open-source help desk ticketing system. In a SOC context, it serves
as the **case management system** — every alert that warrants investigation becomes
a ticket, giving analysts a structured workflow: acknowledge, investigate, document
findings, escalate if needed, and close.

Without a ticketing system, alerts exist only inside the SIEM. A ticketing system
provides accountability (who is working this alert?), documentation (what did we
find?), and metrics (how long did it take to resolve?).

## Infrastructure

- **Platform**: Oracle Cloud Always Free (Ampere A1 Flex, 1 OCPU/2GB RAM)
- **OS**: Ubuntu 22.04 LTS
- **Stack**: Apache + PHP + MariaDB (LAMP)
- **osTicket version**: v1.18.1
- **Public IP**: 141.148.196.50
- **Staff portal**: http://141.148.196.50/scp/
- **User portal**: http://141.148.196.50/

## Installation summary

```bash
# LAMP stack
sudo apt install apache2 php php-cli php-common php-imap php-redis php-snmp \
  php-xml php-zip php-mbstring php-curl php-mysqli php-gd php-intl php-apcu \
  libapache2-mod-php mariadb-server unzip -y

# Database setup
sudo mysql -u root << 'SQLEOF'
CREATE DATABASE osticket;
CREATE USER 'osticket'@'localhost' IDENTIFIED BY 'osticket1234';
GRANT ALL PRIVILEGES ON osticket.* TO 'osticket'@'localhost';
FLUSH PRIVILEGES;
SQLEOF

# Download and extract osTicket
cd /var/www/html
sudo wget https://github.com/osTicket/osTicket/releases/download/v1.18.1/osTicket-v1.18.1.zip
sudo unzip osTicket-v1.18.1.zip -d osticket
sudo cp /var/www/html/osticket/upload/include/ost-sampleconfig.php \
        /var/www/html/osticket/upload/include/ost-config.php
sudo chmod 0666 /var/www/html/osticket/upload/include/ost-config.php
sudo chown -R www-data:www-data /var/www/html/osticket/
```

## Source code fix required (API bug)

osTicket v1.18.1 has a bug in `include/class.api.php` where `getApiKey()` returns
the raw API key string from the HTTP header rather than looking it up from the
database. This causes all API requests to fail with 401 Unauthorized regardless
of whether the key is valid.

Fix applied to `include/class.api.php`:

```php
// Original (broken):
protected function getApiKey() {
    return $_SERVER['HTTP_X_API_KEY'];
}

// Fixed:
protected function getApiKey() {
    if(!isset($_SERVER['HTTP_X_API_KEY']))
        return false;
    return API::lookupByKey($_SERVER['HTTP_X_API_KEY']);
}
```

Also fixed the `requireApiKey()` function to properly honor `0.0.0.0` as
"allow all IPs":

```php
function requireApiKey() {
    if(!($key=$this->getApiKey()))
        return $this->exerr(401, __('Valid API key required'));
    elseif (!$key->isActive())
        return $this->exerr(401, __('API key not active'));
    elseif ($key->getIPAddr() != '0.0.0.0' && $key->getIPAddr() != $_SERVER['REMOTE_ADDR'])
        return $this->exerr(401, __('API key not found/active or source IP not authorized'));
    return $key;
}
```

## Kibana webhook connector configuration

In Kibana → Stack Management → Connectors → Create connector → Webhook:

- **Name**: `osTicket`
- **Method**: POST
- **URL**: `http://141.148.196.50/api/tickets.json`
- **Headers**:
  - `X-API-Key`: `<api-key-from-osticket>`
  - `Content-Type`: `application/json`

**Body template** (used for both SSH and RDP rules):
```json
{
  "alert": true,
  "autorespond": true,
  "source": "API",
  "name": "SOC Alert",
  "email": "soc@gmail.com",
  "subject": "{{rule.name}} - Alert Triggered",
  "ip": "160.191.208.97",
  "message": "data:text/plain,Alert: {{rule.name}}\nSeverity: {{rule.severity}}\nTime: {{date}}\n\nView alerts in Kibana:\nhttp://192.168.10.10:5601/app/security/alerts"
}
```

## Rules with osTicket action attached

- SSH Brute Force detection rule
- RDP Brute Force detection rule (Event ID 4625)

## Confirmed working

End-to-end test: ran SSH brute-force from Kali → detection rule fired in Kibana →
webhook called osTicket API → ticket created automatically in staff portal with
alert details and Kibana link. Ticket creation confirmed via ticket ID returned
by the API.
