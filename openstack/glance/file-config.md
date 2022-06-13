# Tìm hiểu file cấu hình của glance

## Mục lục

[1. File glance-api.conf](#1)

[2. File glance-registry.conf](#2)

[3. File log của glance](#3)

--------

- glance có 2 deamon chạy ẩn là openstack-glnace-api và openstack-glance-api

<a name="1"></a>
### 1. File glance-api.conf

![image](https://user-images.githubusercontent.com/50499526/173267277-88da480c-588a-49e7-91b6-b8146fe8442f.png)

- Path : /etc/glance/glance-api.conf

- cấu hình đến database :

``` connection = mysql+pymysql://glance:GLANCE_DBPASS@controller/glance ```
