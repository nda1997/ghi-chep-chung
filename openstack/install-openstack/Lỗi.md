# Tổng hợp các lỗi trong quá trình cài đặt OPS # 
---------------------------------------------------

![image](https://user-images.githubusercontent.com/50499526/183543720-2129e22c-557b-4d62-9e07-34a79e451d2d.png)

- Vào file /etc/openstack-dashboard/local_settings thêm dòng sau vào cuối file

```
WEBROOT = '/dashboard/'
```

- Lỗi placement-api
Thêm vào file /etc/httpd/conf.d/00-placement-api.conf sau  dòng 15
```
<Directory /usr/bin>
    Require all granted
  </Directory>

```

- ![image](https://user-images.githubusercontent.com/50499526/186562296-5c4eebbf-1090-4fa0-ba95-451a91c8e395.png)

