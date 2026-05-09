# Hệ thống quản lý tiệm cầm đồ

#### Cấu trúc: 
```
├── camdo_django/       # Django project + Dockerfile + requirements.txt
├── myapi/              # API service
├── myweb/              # HTML static site
├── nodered/            # Node-RED flows
├── nginx/              # Nginx config
├── docker-compose.yml  # toàn bộ stack
```

#### Giới thiệu hệ thống
Hệ thống quản lý tiệm cầm đồ được xây dựng bằng:
Django
Docker
phpMyAdmin
Cloudflare Tunnel
MariaDB
Các service chính:django + mariadb + phpmyadmin

### 1. Khởi tạo Ubuntu:
<img width="1483" height="1053" alt="image" src="https://github.com/user-attachments/assets/064e38a6-4e8b-447e-ab6d-f993c8bf8ff2" />

* Cài Ubuntu 24.04 LTS (VM hoặc server).
* Cập nhật hệ thống:

```bash
sudo apt update && sudo apt upgrade -y
```

* Cài Docker & Docker Compose:

```bash
sudo apt install docker.io docker-compose -y
sudo systemctl enable --now docker
```

* Kiểm tra:

```bash
docker --version
docker-compose --version
```

---

## 2️⃣ Tạo cấu trúc thư mục dự án

```text
~/myapp/
├── camdo_django/       # Django project + Dockerfile + requirements.txt
├── myapi/              # API service
├── myweb/              # HTML static site
├── nodered/            # Node-RED flows
├── nginx/              # Nginx config
├── docker-compose.yml  # toàn bộ stack
```

* Tạo folder `camdo_django` cho Django, dễ edit code bằng volume mount.

---

## 3️⃣ Dockerfile Django

* Base image: `python:3.12-bullseye` (để build `mysqlclient` thành công).
* Cài dependencies system (`gcc`, `libmysqlclient-dev`, `libssl-dev`, `libffi-dev`) trước khi pip install.
* Copy `requirements.txt` → pip install → copy Django source.
* Expose port 8000 → chạy server Django dev.

---

## 4️⃣ requirements.txt

```text
Django>=4.2,<5         # framework chính
mysqlclient>=2.2        # connector MariaDB
```

* Giải thích:

  * Django: tạo project, app, models, admin site.
  * mysqlclient: kết nối MariaDB.

---

## 5️⃣ docker-compose.yml – Full stack

* **Services**:

| Service     | Image / Build           | Port | Chức năng                      |
| ----------- | ----------------------- | ---- | ------------------------------ |
| nodered     | nodered/node-red        | 1880 | Workflow / automation          |
| nginx       | nginx:latest            | 80   | Reverse proxy / frontend       |
| cloudflared | cloudflare/cloudflared  | -    | Public Cloudflare Tunnel       |
| filebrowser | filebrowser/filebrowser | 8081 | Quản lý file GUI               |
| myapi       | ./myapi (build)         | -    | API riêng                      |
| db          | mariadb:10.11           | 3307 | Database chính                 |
| phpmyadmin  | phpmyadmin:latest       | 8082 | Kiểm tra DB                    |
| django      | ./camdo_django (build)  | 8000 | Django app quản lý tiệm cầm đồ |

* Volume:

  * db_data → MariaDB persistent
  * mount `./camdo_django:/app` → edit code trực tiếp.

---
Nginx.conf:
<img width="1228" height="695" alt="image" src="https://github.com/user-attachments/assets/d04d9a2b-094a-4157-9152-e9044bab0a00" />


## 6️⃣ Django project setup

* `docker-compose run django python manage.py startproject camdo_proj .`
* Chỉnh **settings.py** để kết nối MariaDB (`db` service, user/password đã config).
* Tạo app `camdo_app` → thêm `models.py` (Customer, PawnItem, FK), `views.py`, template `home.html`.
* Migrate và tạo superuser:

```bash
docker-compose exec django python manage.py migrate
docker-compose exec django python manage.py createsuperuser
```

---

## 7️⃣ Admin site & trang home

* Admin site: thêm/sửa/xóa bảng, FK hiển thị dạng select text.
* Home page (`home_page`) liệt kê **con nợ đến hạn chưa trả tiền**.
* Template sử dụng **Jinja2**:

```html
{% for item in due_items %}
<tr>
  <td>{{ item.customer.name }}</td>
  <td>{{ item.item_name }}</td>
  <td>{{ item.value }}</td>
  <td>{{ item.due_date }}</td>
</tr>
{% endfor %}
```

---

## 8️⃣ PhpMyAdmin

* Dùng để **xem dữ liệu DB** và kiểm chứng FK, không tạo bảng.
* URL: `http://localhost:8082/`
* Sơ lược bảng viết tay
<img width="1920" height="2560" alt="image" src="https://github.com/user-attachments/assets/686e69d9-7880-4bc1-b0df-b1b77e7ce542" />

---

## 9️⃣ Cloudflare Tunnel
<img width="1919" height="951" alt="image" src="https://github.com/user-attachments/assets/6fabc1a3-39a9-4177-bf59-7ae6f617ce94" />

* Tạo subdomain: `camdo.anhtu.divu.click`
* DNS: CNAME → `divu.click`, **proxy bật**.
* Trong Zero Trust → Tunnels → Public Hostname:

| Hostname               | Service                                                              | TLS  |
| ---------------------- | -------------------------------------------------------------------- | ---- |
| camdo.anhtu.divu.click | [http://host.docker.internal:8000](http://host.docker.internal:8000) | Auto |

* Tunnel route request từ internet → host port 8000 → Django container.

---

## 🔟 Kết quả

* Home page hiển thị danh sách con nợ đến hạn.
* Admin site quản lý dữ liệu các bảng, FK hiển thị text.
* PhpMyAdmin kiểm chứng CS.
* Tất cả chạy trên Docker, public qua Cloudflare Tunnel.

---

 










