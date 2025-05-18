# filebeat
1. Filebeat 설치 (Ubuntu 22.04)

```bash
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.6.2-amd64.deb
sudo dpkg -i filebeat-8.6.2-amd64.deb
```

2. Filebeat 설정 파일 수정

```bash
sudo nano /etc/filebeat/filebeat.yml
```

아래 항목을 다음처럼 수정:

```yaml
filebeat.inputs:
  - type: log         # 이 부분이 원래 filestream이였는데 내가 임의로 log로 바꿈
    enabled: true
    paths:
      - /var/log/*.log
      - /var/log/syslog       # 여긴 추가 안해도 되는데.. 불안해서 걍 했음
      - /var/log/auth.log
```

 `filestream` 관련 설정은 모두 주석 처리하거나 제거했음 이유는 나처럼 초보자가 filestream 쓰면 어디서 오류났는지 확인하고 고치기 복잡해서 걍 간단하게 log로 바꿨음

 

3. Filebeat 시스템 로그 모듈 활성화 (선택)

```bash
sudo filebeat modules enable system
```

4. Filebeat 기본 설정 및 대시보드 설치

```bash
sudo filebeat setup
```

> 이 과정에서 Kibana와 연동된 인덱스 템플릿, ingest pipeline, dashboard가 자동 설치됨
> 

5. Filebeat 실행 및 서비스 시작

```bash
sudo systemctl start filebeat
sudo systemctl enable filebeat
```

6. 테스트 로그 삽입 (로그 수집 확인용)

```bash
echo "filebeat test log $(date)" | sudo tee -a /var/log/syslog
```

7. Filebeat 상태 확인

```bash
sudo journalctl -u filebeat --no-pager | tail -n 50
```

정상이라면 `"Harvester started for file"` 같은 로그가 떠야 함

 

8. Kibana에서 로그 확인

1. `http://localhost:5601` 접속
2. 왼쪽 메뉴 → Discover
3. Index Pattern으로 `filebeat-*` 선택
4. 우측 상단 시간 필터를 Today 또는 Last 1 hour로 변경
5. 로그 확인 성공 

---

 9. Elasticsearch에서 직접 확인 (선택)

```bash
curl -k -u elastic:p@ssw0rd1234 "https://localhost:9200/filebeat-*/_search?pretty"
```

옵션 -k는 ssl인증서가 없어도 걍 무시하고 curl 명령 던지는거임

![image](https://github.com/user-attachments/assets/20a42756-746e-41c4-a927-b8869dacdb5d)
