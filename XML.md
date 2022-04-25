# Tìm hiểu về file XML trong KVM

### Mục lục

[1. Tổng quan về file XML](#1)

[2. Các thành phần của file XML](#2)

<a name="1"></a>

## 1, Tổng quan về file XML
- VM trong KVM có 2 thành phần chính là file xml và file images.
- File XML được tạo ra để mô tả lại cấu trúc và các tính năng khác của 1 đối tượng cụ thể 
- XML trong KVM  có chứa thông tin của máy ảo : ram, cpu, disk , các thiết bị I/O ,...
- Thư viện libvirt sẽ dùng file xml này cùng các thông số trong đó để  khởi chạy 1 tiến trình chạy máy ảo

![image](https://user-images.githubusercontent.com/50499526/165048585-2035056b-7cdf-4df5-8661-57a7aa6f5937.png)

<a name="2"></a>
## 2, Các thành phần của file XML
- File xml nhìn cơ bản có thể thấy tổ chức theo khối lệnh, có nhiều khối lệnh cùng trong 1 khối lệnh tổng quan
- Khối lệnh chính trong kvm là thẻ ' domain type '
   + Tham số `type` cho biết hypervisor đang sử dụng của VM.
   + `name` : tên máy ảo 
   + `uuid` : mã nhận diện , mỗi máy ảo chỉ có duy nhất 1 uuid riêng biệt giống như mac-address .
   + `memory unit` : mặc định là Kib (kibibytes ) chỉ dung lượng ram của máy ảo
   + `currentmemory` : dung lượng ram được sử dụng tại thời điểm xuất file xml
   + `vcpu` : số cpu cấp phát cho máy ảo 
   + thẻ `device` : cung cấp các thiết bị I/O
       + `disk` : thông tin lưu trữ ổ đĩa của VM 

```sh
<disk type='file' device='cdrom'>
      <driver name='qemu' type='raw'/>
      <target dev='hdb' bus='ide'/>
      <readonly/>
      <address type='drive' controller='0' bus='0' target='0' unit='1'/>
    </disk>
```
   + thẻ `controller` : cung cấp các 
   + `interface`: cung cấp các card mạng vật lí ảo của VM   ( ví dụ như type kết nối , mac address , ...)
```sh
<interface type='bridge'>
      <mac address='52:54:00:e2:6b:f6'/>
      <source bridge='vlan268'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x0a' function='0x0'/>
    </interface>
```

   +  `clock`: Thiết lập về thời gian

```sh
<clock offset='utc'>
    <timer name='rtc' tickpolicy='catchup'/>
    <timer name='pit' tickpolicy='delay'/>
    <timer name='hpet' present='no'/>
</clock>
```



### Note :  Khi ta thay đổi tham số trong file xml thì việc thay đổi chưa thực hiện ngay được . Ta cần define lại VM và khởi động lại máy ảo nếu cần thiết  
```sh
virsh define <file xml máy ảo  > 
``` 
Có thể tìm thấy file thông tin máy ảo trong thư mục /etc/libvirt/qemu 

