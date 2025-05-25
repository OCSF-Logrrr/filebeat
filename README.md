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
  - type: log
    enabled: true
    paths:
      - /var/log/syslog
      - /var/log/auth.log

output.logstash:
  hosts: ["Logstash IP주소:5044"]    #❗❗ 반드시 수정하세요 ❗❗
  ssl.enabled: false
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
- Kibana 접속 (보통 https://localhost:5601)

- Discover → filebeat-* 인덱스 선택

- 수집된 로그 확인
