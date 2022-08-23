# Ansible AWX
## 1. Các thành phần trong Ansible AWX
- `AWX engine`: vai trò engine thực thi các thành phần trong Ansible.
- `NGINX`: vai trò web server cung cấp giao diện web cho người dùng.
- `PostgreSQL`: vai trò database lưu trữ dữ liệu khởi tạo, dữ liệu log sinh ra trong quá trình Ansible thực thi.
- `Redis`: vai trò cache server.

## 2. Sử dụng Ansible AWX để thực thi các Playbook
- `Inventory`
- `Credentials`: là nơi lưu trữ các thông tin bảo mật của AWX. Ví dụ ssh private key để ssh đến các host, mật khẩu các tài khoản.
- `Projects`: là repository lưu trữ các tài nguyên của Ansible Playbook (Roles, Playbooks).
    + Một project có thể lưu một hoặc nhiều các playbook sử dụng cho một mục đích hoặc đối tượng.
- `Template`: là nơi khai báo các thực thi các Playbook.
