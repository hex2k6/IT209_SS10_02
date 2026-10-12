# Bài 2: Đóng gói Web Backend bằng Single-stage Dockerfile

## 1. Ứng dụng và cấu trúc

Ứng dụng Node.js/Express có endpoint `GET /` trả HTTP 200 và JSON:

```json
{"status":"success","message":"Hello from Docker Single-stage!"}
```

Các tệp nộp bài:

- `server.js`: mã nguồn API; lắng nghe trên `0.0.0.0` và cổng từ biến môi trường `PORT`.
- `package.json`: khai báo Express và lệnh `npm start`.
- `package-lock.json`: khóa phiên bản phụ thuộc để cài đặt bằng `npm ci`.
- `Dockerfile`: đóng gói ứng dụng với một stage.
- `.dockerignore`: loại trừ các tệp không cần thiết khỏi build context.
- `README.md`: hướng dẫn và kết quả thực hành.

## 2. Dockerfile

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --omit=dev
COPY server.js ./
ENV PORT=3000
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

Dockerfile có đúng một chỉ thị `FROM`, do đó là single-stage. Dùng base image `node:18-alpine` đúng ràng buộc đề bài. `WORKDIR` đặt thư mục làm việc `/app`. Copy manifest và lockfile trước khi cài phụ thuộc giúp tái sử dụng cache khi chỉ sửa mã nguồn. `RUN npm ci --omit=dev` cài các phụ thuộc chạy ứng dụng theo lockfile. `USER node` chạy tiến trình ứng dụng bằng user không phải root trong container.

`ENV PORT=3000` đặt cổng mặc định, `EXPOSE 3000` mô tả cổng ứng dụng; cần `-p 8080:3000` khi chạy để publish cổng ra host. `CMD` dùng exec form theo yêu cầu, chạy trực tiếp `node server.js`.

## 3. Các lệnh thực hành

Chạy trong thư mục `homework/session_10/ex2/`:

```bash
docker build -t hw-backend:v1 .
docker run -d --name app-single-stage -p 8080:3000 hw-backend:v1
docker ps
docker logs app-single-stage
curl -i http://localhost:8080/
```

Trên PowerShell của máy thực hành, dùng `curl.exe -i http://localhost:8080/` để gọi chương trình curl trực tiếp.

Cổng 8080 và tên `app-single-stage` cần chưa được sử dụng trước khi chạy. Nếu cần chạy lại bài này, dừng và xóa container cũ của bài bằng `docker stop app-single-stage` rồi `docker rm app-single-stage`, sau đó chạy lại lệnh `docker run`. Các lệnh dọn dẹp này là hướng dẫn, chưa thực hiện trong lần kiểm tra bên dưới.

## 4. Kết quả thực tế

Môi trường: PowerShell trên Windows, Docker Desktop Linux Engine. Toàn bộ đầu ra bên dưới lấy từ lần chạy thật; build và curl đều có mã thoát `0`.

### Build image

```text
PS> docker build -t hw-backend:v1 .
#0 building with "desktop-linux" instance using docker driver

#1 [internal] load build definition from Dockerfile
#1 transferring dockerfile: 214B 0.0s done
#1 DONE 0.1s

#2 [internal] load metadata for docker.io/library/node:18-alpine
#2 DONE 0.1s

#3 [internal] load .dockerignore
#3 transferring context: 96B done
#3 DONE 0.0s

#4 [internal] load build context
#4 transferring context: 30.10kB 0.0s done
#4 DONE 0.0s

#5 [1/5] FROM docker.io/library/node:18-alpine@sha256:8d6421d663b4c28fd3ebc498332f249011d118945588d0a35cb9bc4b8ca09d9e
#5 resolve docker.io/library/node:18-alpine@sha256:8d6421d663b4c28fd3ebc498332f249011d118945588d0a35cb9bc4b8ca09d9e 0.0s done
#5 DONE 0.1s

#6 [2/5] WORKDIR /app
#6 DONE 0.0s

#7 [3/5] COPY package.json package-lock.json ./
#7 DONE 0.0s

#8 [4/5] RUN npm ci --omit=dev
#8 2.490
#8 2.490 added 68 packages, and audited 69 packages in 2s
#8 2.490
#8 2.490 16 packages are looking for funding
#8 2.490   run `npm fund` for details
#8 2.491
#8 2.491 found 0 vulnerabilities
#8 2.493 npm notice
#8 2.493 npm notice New major version of npm available! 10.8.2 -> 12.2.0
#8 2.493 npm notice Changelog: https://github.com/npm/cli/releases/tag/v12.2.0
#8 2.493 npm notice To update run: npm install -g npm@12.2.0
#8 2.493 npm notice
#8 DONE 2.6s

#9 [5/5] COPY server.js ./
#9 DONE 0.0s

#10 exporting to image
#10 exporting layers
#10 exporting layers 0.4s done
#10 exporting manifest sha256:3b1b315836bd4ccce248ee059b7df16aae47ec614619377b560aa2d010084f3b 0.0s done
#10 exporting config sha256:922256e53031c79af2d7d31d22f37106ca78a1e37067004bafa7c7034e59a082 0.0s done
#10 exporting attestation manifest sha256:99a510e9257f504ca90605cf268e7ca5ec72e857b6465c00c752b528b569e3b6 0.0s done
#10 exporting manifest list sha256:af630e9c55d70fbdf0d0a460ec2f58703a927287551ee4a76e0471ea0e92d9f1 0.0s done
#10 naming to docker.io/library/hw-backend:v1 done
#10 unpacking to docker.io/library/hw-backend:v1
#10 unpacking to docker.io/library/hw-backend:v1 0.5s done
#10 DONE 1.1s
```

### Khởi chạy container

```text
PS> docker run -d --name app-single-stage -p 8080:3000 hw-backend:v1
f54a7e4857dd2bdab4dac700563bee64727a7964c80173b508d313c634ae4b9c
```

### Danh sách container

```text
PS> docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                                         NAMES
f54a7e4857dd   hw-backend:v1   "docker-entrypoint.s…"   14 seconds ago   Up 13 seconds   0.0.0.0:8080->3000/tcp, [::]:8080->3000/tcp   app-single-stage
```

Container hiển thị trạng thái `Up`, ánh xạ cổng host 8080 tới cổng 3000 trong container.

### Log ứng dụng

```text
PS> docker logs app-single-stage
Server listening on port 3000
```

### Kiểm tra HTTP

```text
PS> curl.exe -i http://localhost:8080/
HTTP/1.1 200 OK
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 64
ETag: W/"40-upZb7Fps+hszzRU9BoKtiCsmGCQ"
Date: Mon, 12 Oct 2026 16:37:21 GMT
Connection: keep-alive
Keep-Alive: timeout=5

{"status":"success","message":"Hello from Docker Single-stage!"}
```

Đã kiểm tra mã HTTP là `200 OK`, Content-Type là JSON và phân tích body xác nhận hai trường `status` và `message` đúng mẫu. Kiểm tra thêm bằng `docker inspect --format '{{.State.Running}}' app-single-stage` trả về `true`.

## 5. Kết luận

Hoàn thành Dockerfile single-stage với đầy đủ `FROM`, `WORKDIR`, `COPY`, `RUN`, `ENV`, `EXPOSE`, `CMD`. Image `hw-backend:v1` build thành công, container `app-single-stage` đang chạy và API trả HTTP 200 cùng JSON đúng yêu cầu. Giữ container chạy để có thể truy cập lại tại `http://localhost:8080/`.