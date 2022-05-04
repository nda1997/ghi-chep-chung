# Create VM by CLI

## 1. Tạo máy ảo với lệnh virsh
Để tạo máy ảo bằng command line ta dùng lệnh `virt-install`

```
$ virt-install \
> --name=anhnd-kvm01 \
> --vcpus=1 \
> --memory=1024 \
> --location=CentOS-7-x86_64-Minimal-1804.iso \
> --disk=/var/lib/libvirt/images/,size=10 \
> --os-variant=rhel7 \
> --network bridge=virbr1
```

Trong đó 
 * `--name` đặt tên cho máy ảo định tạo
 * `--vcpus` là tổng số CPU đinhj tạo cho máy ảo
 * `--memory` chỉ ra dung lượng RAM cho máy ảo (tính bằng MB)
 * `--location`  chỉ ra đường dẫn đến file (có thể là đường dẫn trên internet).
 * `--disk` chỉ ra vị trí lưu disk của máy ảo. `size` chỉ ra dung lượng disk của máy ảo(tính bằng GB)
 * `--os-variant` chỉ ra kiểu của HĐH của máy ảo đang tạo. Option này có thể chỉ ra hoặc không nhưng nên sử dụng nó vì nó sẽ cải thiện hiệu năng của máy ảo. Nếu bạn không biết HĐH hành của mình thuộc loại nào bạn có thể tìm kiếm thông tin bằng cách dùng lệnh `osinfo-query os`
 * `--network` chỉ ra cách kết nối mạng của máy ảo.
Trên đây là một số option cơ bản để tạo máy ảo. Bạn có thể tìm hiểu thêm bằng cách sử dụng lệnh `virt-install --help`


- Hiển thị danh sách máy ảo 
``` Ta dùng lệnh virsh list --all ```
- Tắt máy ảo  
``` ta dùng lệnh virsh shutdown tên_máy_ảo ```
- Bật máy 
``` Dùng lệnh virsh start tên_máy ```
- Xóa máy ảo 
``` ta dùng lệnh virsh undefine tên_máy_ảo ```
#### Lưu ý nên tắt máy ảo khi thực hiện lênh xóa . với lệnh này sẽ xóa toàn bộ thông tin liên quan đến máy ảo

- ``` virsh edit Tên_VM ``` : sửa thông tin phần cứng máy ảo 
- ``` qemu-img info file-disk  : xem thông tin chi tiết về disk máy ảo ```
- ``` virsh dominfo tên-VM  : xem thông tin máy áo ```

