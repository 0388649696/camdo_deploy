# camdo_deploy

#### Cấu trúc: 
```
camdo_django/
├── Dockerfile              # Dockerfile cho Django container
├── requirements.txt        # Thư viện Python cần thiết (Django, mysqlclient)
├── camdo_proj/             # Django project
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py         # Chỉnh DB, INSTALLED_APPS, template dirs
│   ├── urls.py             # URL routing
│   └── wsgi.py
├── camdo_app/              # Django app quản lý tiệm cầm đồ
│   ├── __init__.py
│   ├── admin.py            # Đăng ký models để hiển thị admin
│   ├── apps.py
│   ├── models.py           # Customer, PawnItem, FK, các bảng nghiệp vụ
│   ├── views.py            # home_page, liệt kê con nợ đến hạn
│   ├── urls.py             # App-specific urls
│   └── templates/
│       └── home.html       # Template Jinja2 cho home_page
├── manage.py               # Script chạy Django CLI
└── db.sqlite3 (nếu dev thử bằng SQLite) hoặc dùng MariaDB container
```

#### Giới thiệu hệ thống
Hệ thống quản lý tiệm cầm đồ được xây dựng bằng:
Django
Docker
phpMyAdmin
Cloudflare Tunnel
MariaDB
Các service chính:django + mariadb + phpmyadmin











