# 🐧 Filebeat 8.6.2 설치 및 Logstash 연동 가이드 (Linux)

이 문서는 **리눅스 서버에 Filebeat 8.6.2를 설치**하고, **Logstash로 로그를 전송**하는 방법을 안내합니다.

---

## 📥 1. Filebeat 8.6.2 다운로드 및 설치
```
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.6.2-amd64.deb
sudo dpkg -i filebeat-8.6.2-amd64.deb
```

설치 후 기본 경로:

- 실행 파일: /usr/bin/filebeat
  
- 설정 파일: /etc/filebeat/filebeat.yml
  
- 서비스 파일: systemctl 사용 가능

## 📝 2. filebeat.yml 설정
```
filebeat.inputs:

# 1. ModSecurity
- type: log
  id: modsecurity
  enabled: false
  paths:
    - /var/log/modsec_audit.log
  fields:
    service: modsecurity
    source_type: waf
    event.dataset: modsecurity.audit
  fields_under_root: true

# 2. BIND9
- type: log
  id: bind9
  enabled: false
  paths:
    - /var/log/named/query.log
    - /var/log/named/named.log
  fields:
    service: bind9
    source_type: dns
    event.dataset: bind9.dns
  fields_under_root: true

# 3. Nginx
- type: log
  id: nginx
  enabled: false
  paths:
    - /var/log/nginx/access.log
    - /var/log/nginx/error.log
  fields:
    service: nginx
    source_type: web_server
    event.dataset: nginx.access
  fields_under_root: true

# 4. UFW
- type: log
  id: ufw
  enabled: false
  paths:
    - /var/log/ufw.log
  fields:
    service: ufw
    source_type: firewall
    event.dataset: ufw.firewall
  fields_under_root: true

# 5. Ubuntu System Logs
- type: log
  id: ubuntu
  enabled: false
  paths:
    - /var/log/syslog
    - /var/log/auth.log
    - /var/log/kern.log
    - /var/log/dmesg
    - /var/log/dpkg.log
    - /var/log/alternatives.log
    - /var/log/faillog
    - /var/log/lastlog
    - /var/log/wtmp
    - /var/log/btmp
  fields:
    service: ubuntu
    source_type: os
    event.dataset: ubuntu.syslog
  fields_under_root: true

# 6. Apache
- type: log
  id: apache
  enabled: true
  paths:
    - /var/log/apache2/access.log*
      # - /var/log/apache2/error.log*
  fields:
    service: apache
    source_type: web_server
    event.dataset: apache.access
  fields_under_root: true

# 7. PHP
- type: log
  id: php
  enabled: false
  paths:
    - /var/log/php/php_errors.log
  fields:
    service: php
    source_type: app_server
    event.dataset: php.errors
  fields_under_root: true
  multiline.pattern: '^\['
  multiline.negate: true
  multiline.match: after

# 8. MySQL
- type: log
  id: mysql
  enabled: false
  paths:
    - /var/log/mysql/error.log
    - /var/log/mysql/general.log
  fields:
    service: mysql
    source_type: database
    event.dataset: mysql.general
  fields_under_root: true

# 9. MongoDB
- type: log
  id: mongodb
  enabled: false
  paths:
    - /var/log/mongodb/mongod.log
  fields:
    service: mongodb
    source_type: database
    event.dataset: mongodb.logs
  fields_under_root: true

# ============================= 프로세서 설정 =============================

processors:
  - add_host_metadata:
      when.not.contains.tags: forwarded
  - add_cloud_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~

# ============================= Kafka 출력 설정 =============================

output.kafka:
  hosts: ["221.144.36.127:9092"]
  topic: "raw-logs"
  partition.round_robin:
    reachable_only: true
  codec.json:
    pretty: false
  required_acks: 1
  compression: gzip
  max_message_bytes: 1000000
```

## ▶️ 3. Filebeat 서비스 실행
```
# 설정 테스트
sudo filebeat test config

# 서비스 등록 및 시작
sudo systemctl enable filebeat
sudo systemctl start filebeat

# 상태 확인
sudo systemctl status filebeat
```
## 🧪 4. Kibana에서 데이터 확인
- Kibana 접속 (보통 http://localhost:5601)

- Discover → filebeat-* 인덱스 선택

- 수집된 로그 확인


## 🪵 5. Kafka 수신 로그 확인 명령어
Kafka 브로커에서 Filebeat가 전송한 로그가 도착했는지 확인하려면:
```
docker exec -it kafka1 kafka-console-consumer \
  --topic raw-logs --bootstrap-server kafka1:9092 \
  --from-beginning --timeout-ms 10000
```
>⚠️ 로그가 없을 경우 Filebeat 출력 로그 또는 네트워크 연결 상태 확인 필요

## 📉 6. Filebeat 자체 로그 경로
에러 발생 시 다음 로그를 확인하면 유용합니다:
```
cat /var/log/filebeat/filebeat
```
또는 systemd 사용 시:
```
journalctl -u filebeat -f
```

## ⚠️ 7. 자주 발생하는 문제 & 대응
| 문제           | 원인                      | 해결 방법                          |
| ------------ | ----------------------- | ------------------------------ |
| Kafka 전송 실패  | Kafka 브로커 접근 불가, 인증 오류  | 브로커 IP, 방화벽, 포트 확인             |
| 로그 수집 안 됨    | `enabled: false`, 경로 오타 | `enabled: true` 확인, `paths` 확인 |
| Kibana에 안 보임 | 인덱스 미생성                 | `filebeat-*` 인덱스 패턴 수동 생성      |

## ✅ 추가 팁
- filebeat.yml은 변경 후 반드시 sudo filebeat test config로 구문 검사
- /var/log/apache2/access.log* 같이 와일드카드 사용 시 rotate된 파일도 수집 가능

