# ========== 빌드 환경 ==========

# 베이스 이미지 선택
FROM node:alpine as builder

# 작업 디렉터리 지정
WORKDIR '/app'

# 리액트 프로젝트 이미지 안으로 복사
COPY react-nginx-app/ .

# 의존성 모듈 설치
RUN npm install

# 빌드
RUN npm run build

# ========== 프로덕션 환경 ==========
FROM nginx

# 빌드환경에서 빌드 결과를 프로덕션 환경인 nginx 안으로 복사
COPY --from=builder /app/build /usr/share/nginx/html