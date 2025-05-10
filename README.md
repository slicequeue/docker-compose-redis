# Docker Compose Redis

## 프로젝트 소개
Docker Compose를 사용하여 Redis 환경을 빠르게 구성하고 실행할 수 있는 설정

---

## 요구사항
- Docker (v20.10 이상 권장)
- Docker Compose (v2.0 이상 권장)

---

## Docker Compose 설정 파일

이 프로젝트의 `docker-compose.yml` 파일은 Redis를 설정하기 위해 아래와 같은 구성을 포함합니다:

```yaml
version: '3.7'
services:
    redis:
      image: redis:alpine
      container_name: redis-container
      hostname: redis_boot
      restart: always
      labels:
        - "name=redis"
        - "mode=standalone"
      ports:
        - 6379:6379
      volumes:
        - ./data/:/data
        - ./conf/redis.conf:/usr/local/etc/redis/redis.conf:ro
      env_file:
        - ./.env
      command: >
        sh -c "redis-server --port 6379
              --requirepass ${REDIS_PASSWORD}
              --save 900 1
              --save 300 10
              --save 60 10000
              --dbfilename dump.rdb
              --appendonly yes
              --appendfilename appendonly.aof
              --appendfsync everysec
              --auto-aof-rewrite-percentage 100
              --auto-aof-rewrite-min-size 64mb"
```

---

### 주요 설정 옵션 설명

1. **`save` 옵션:**
   - Redis 데이터베이스 상태를 주기적으로 저장하는 스냅샷(Snapshot) 기능을 설정합니다.
   - 형식: `save <초 단위> <변경 횟수>`
   - 예시:
     - `save 900 1`: 900초(15분)마다 데이터가 1회 변경되면 스냅샷 저장.
     - `save 300 10`: 300초(5분)마다 데이터가 10회 변경되면 스냅샷 저장.
     - `save 60 10000`: 60초(1분)마다 데이터가 10,000회 변경되면 스냅샷 저장.
   - 스냅샷 파일은 `dump.rdb`로 저장됩니다.

2. **`appendonly` 옵션:**
   - Redis의 데이터 지속성을 위해 명령어 로그를 저장하는 AOF(Append-Only File) 기능을 활성화합니다.
   - `appendonly yes`: AOF 기능 활성화.
   - `appendfilename appendonly.aof`: AOF 파일 이름을 지정합니다.
   - AOF 파일은 Redis 서버가 종료되더라도 모든 명령 실행 기록을 보존합니다.

3. **`appendfsync` 옵션:**
   - AOF 파일 동기화 주기를 설정합니다.
   - 예시:
     - `appendfsync everysec`: 1초마다 AOF 파일을 디스크에 동기화(권장).
     - `appendfsync always`: 모든 명령 실행 후 즉시 동기화(성능 저하 가능).
     - `appendfsync no`: 동기화 비활성화(데이터 손실 위험 있음).

4. **`auto-aof-rewrite-percentage` 및 `auto-aof-rewrite-min-size`:**
   - AOF 파일 자동 재작성 조건을 설정합니다.
   - `auto-aof-rewrite-percentage 100`: AOF 파일 크기가 이전 크기의 100% 증가 시 재작성.
   - `auto-aof-rewrite-min-size 64mb`: AOF 파일 크기가 최소 64MB 이상일 경우에만 재작성.

5. **`requirepass` 옵션:**
   - Redis 서버에 접속하기 위한 비밀번호를 설정합니다.
   - `.env` 파일에서 `REDIS_PASSWORD` 환경 변수를 설정하면 해당 비밀번호가 적용됩니다.

---

## 설치 및 실행

1. 이 레포지토리를 클론합니다:
   ```bash
   git clone https://github.com/slicequeue/docker-compose-redis.git
   cd docker-compose-redis
   ```

2. `.env` 파일을 생성합니다:
   `.env` 파일에 Redis 비밀번호를 설정합니다.
   ```env
   REDIS_PASSWORD=your_redis_password
   ```

3. Docker Compose를 사용하여 Redis를 실행합니다:
   ```bash
   docker-compose up -d
   ```

4. Redis가 실행 중인지 확인합니다:
   ```bash
   docker ps
   ```

---

## 사용법

- **Redis CLI 접속:**
  ```bash
  docker exec -it redis-container redis-cli -a your_redis_password
  ```

- **데이터 확인:**
  Redis CLI에서 간단한 명령어를 실행하여 데이터베이스 상태를 확인할 수 있습니다:
  ```bash
  set key1 "Hello, Redis!"
  get key1
  ```

- **컨테이너 중지 및 제거:**
  컨테이너를 중지하려면 다음 명령어를 실행하세요:
  ```bash
  docker-compose down
  ```

---

## 파일 구성
- `docker-compose.yml`: Redis 환경 설정 파일입니다.
- `.env`: Redis 비밀번호를 저장하는 환경 변수 파일입니다.
- `./data/`: Redis 데이터가 저장되는 디렉토리입니다.
- `./conf/redis.conf`: Redis 설정 파일입니다.

---
