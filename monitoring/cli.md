Để xem các tiến trình sử dụng phần trăm (%) RAM nhiều nhất, ta sử dụng lệnh sau:
-  ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem

- ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head ( 9 tiến trình sử dụng nhiều ram nhất )

Để theo dõi liên tục theo thời gian, dữ liệu cập nhật 1 giây 1 lần, ta dùng lệnh

- watch -n 1 'ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem | head'

Xem 5 tiến trình sử dụng nhiều CPU nhất:

- ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head -n 6

Để theo dõi liên tục với dữ liệu cập nhật 1 giây 1 lần:
watch -n 1 'ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head -n 6'

theo dõi thông số sử dụng RAM và CPU của tiến trình Mysqld

 -  watch -n 1 'ps -eo pid,ppid,cmd,%mem,%cpu | egrep "mysqld"'
