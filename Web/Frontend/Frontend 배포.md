## 목표: Jenkins를 통한 Frontend 자동 배포

1. GitHub/main에 코드 push 발생
2. Jenkins가 `frontend`를 빌드
3. `nginx/html/`에 정적 파일 복사
4. 필요 시 `nginx` 이미지도 다시 빌드
5. 컨테이너 재시작 or 롤링 배포

### 각 단계

|단계|내용|
|---|---|
|`frontend-prod` 빌드|Dockerfile에서 의존성 설치, 소스 복사까지만 함|
|`frontend-prod` 실행|`command`에서 `pnpm build && cp -r out/* /out` 수행|
|`nginx/html`에 정적 파일 복사됨|이 경로가 `nginx` Dockerfile에서 `COPY` 되므로 포함됨|
|`nginx` 이미지 빌드|최신 정적 파일 포함하여 새 이미지 생성|
|`nginx` 컨테이너 재시작|사용자에게 최신 빌드 서비스 제공|

### 실제 동작 과정

1. `Jenkins`에서 `git pull` → `docker compose build frontend-prod`
2. `docker compose run --rm frontend-prod` → `out/` 결과 생성
3. `nginx/html/` 경로에 정적 파일이 복사됨
4. `docker compose build nginx` → 최신 정적 파일 포함한 `nginx` 이미지 생성
5. `docker compose up -d nginx` → 배포 완료

## 설정
### 1. frontend/Dockerfile
```Dockerfile
# frontend/Dockerfile
FROM node:20-alpine

WORKDIR /app

# 1. pnpm 설치
RUN npm install -g pnpm

# 2. 의존성 설치
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile

# 3. 소스 복사 (실제 빌드는 compose에서 진행)
COPY . .

# 4. 기본 종료 명령 (컨테이너 실행 시 종료)
CMD ["true"]
```
- 빌드는 하지 않고, **이미지 안에 준비된 상태**로 유지 → 빠른 실행을 위해 레이어 캐시 활용 가능

### 2. docker-compose.yml
```yaml
version: "3.9"

services:
  frontend-prod:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: frontend-prod
    volumes:
      - ./nginx/html:/out            # ✅ 결과물이 저장될 경로
    command: >
      sh -c "pnpm build && cp -r out/* /out"
    restart: "no"

  nginx:
    build:
      context: ./nginx
      dockerfile: Dockerfile
    container_name: nginx
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - frontend-prod
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```
- `frontend-prod` 실행 시 `pnpm build && cp -r out/* /out` 을 수행 → `nginx/html`에 정적 파일이 복사됨

### 3. `next.config.mjs`
```js
export default {
  output: 'export',
  distDir: 'out',         // 프로젝트 루트에 out/ 생성되도록 설정
  eslint: { ignoreDuringBuilds: true },
  typescript: { ignoreBuildErrors: true },
  images: { unoptimized: true },
}
```

### 4. nginx/Dockerfile
```Dockerfile
# nginx/Dockerfile

FROM nginx:1.27-alpine

# Nginx 설정 파일 복사 (React/Next SPA 대응 포함)
COPY ./conf.d/default.conf /etc/nginx/conf.d/default.conf

# SSL 인증서 (certbot 등으로 발급 받은 경우)
COPY ./ssl /etc/nginx/ssl

# 정적 빌드 결과물 복사 (frontend-prod가 ./nginx/html에 복사해둔 상태)
COPY ./html /usr/share/nginx/html
```

### 5. 실행 과정

1. `frontend-prod` 실행: `docker compose run --rm frontend-prod`
2. `nginx` 이미지 빌드: `docker compose build nginx`
3. 배포: `docker compose up -d nginx`

### 6. Jenkins 설정
- `frontend-prod`만 다시 빌드하면 최신화된 정적 빌드 결과물이 `nginx`에 마운트한 디렉토리에 갱신되어, `frontend-prod` 컨테이너만 재실행 시키기만 하면 됨