# SECTION 01

```docker
# 내부에서 nodejs사용가능
From node:14
# 작업하고자 하는 디렉토리
WORKDIR /app
# package.json 카피
COPY package.json .
# 애플리케이션에 필요한 모든 종속성 설치
RUN npm install
# 나머지 코드를 여기 복사
COPY . .
# 포트3000을 외부에 노출
EXPOSE 3000
# node명령으로 app.mjs실행
CMD ["node", "app.mjs"]
```

## 도커 순서

1. 터미널에서 도커 빌드  
   이미 존재하는 노드 환경을 Docker Hub에서 다운로드

   ```
   docker build .
   ```

2. 만들어진 이미지의 ID를 얻는다.

   ```
   => exporting to image
   => => exporting layers
   => => writing image sha256:7da377cb6aabab88f67ca3b9203efe516c8e86c0f2a1387290caaf5788c1001a
   ```

3. 만들어진 이미지 기반으로 컨테이너 실행  
    실제로 실행하려는 컨테이너에 해당 포트를 게시해야한다.  
    로컬시스템의 로컬호스트를 사용하여 컨테이너 대신 포트 3000에서 실행되는 애플리케이션에 연결

   ```
   docker run -p 3000:3000 7da377cb6aabab88f67ca3b9203efe516c8e86c0f2a1387290caaf5788c1001a
   ```

4. localhost:3000에 접속가능

5. 새로운 터미널을 열어 docker ps를 입력하여 실행중인 모든 컨테이너를 가져온다.

6. docker stop [종료할 컨테이너 이름] 으로 컨테이너를 중지하고 종료시킨다.
