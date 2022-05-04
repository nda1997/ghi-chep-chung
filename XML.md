# Tìm hiểu về file XML trong KVM

### Mục lục

[1. Tổng quan về file XML](#1)

[2. Các thành phần của file XML](#2)

[3. Tạo VM từ file xml](#3)

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
   +`fearture` : là hypervisors cho phép thao tác một số tính năng như bật /tắt
```sh
<features>
    <acpi/>
    <apic/>
  </features>
```
         + `acpi`: sử dụng quản lý máy ảo ( ví dụ như bắt buộc tắt máy ảo )
         + `apic` : bộ điều khiển ngắt 

### Note :  Khi ta thay đổi tham số trong file xml thì việc thay đổi chưa thực hiện ngay được . Ta cần define lại VM và khởi động lại máy ảo nếu cần thiết  
```sh
virsh define <file xml máy ảo  > 
``` 
Có thể tìm thấy file thông tin máy ảo trong thư mục /etc/libvirt/qemu 

<a name="3"></a>

## 3, Tạo VM từ file xml

Bước 1 : tạo mã uuid
+ Tạo mã uuid, cài đặt gói uuid và generate đoạn mã uuid
 
```sh
yum install uuid -y

uuid
```

![image](https://user-images.githubusercontent.com/50499526/165058510-ff424765-fd1e-4582-9fd9-dac5c2a985cc.png)

Bước 2: Tạo disk
+ Tạo một ổ đĩa cho máy ảo khai báo dung lượng và định dạng là qcow2

```sh
qemu-img create -f raw /var/lib/libvirt/images/anhnd-kvm01.qcow2 15G
```

Bước 3 : Viết file xml

- Tham số cơ bản  

```sh
+ Thông tin RAM, vCPU, disk

+ Đường dẫn tới disk: /var/lib/libvirt/images/anhnd-kvm01.qcow2

+ Máy ảo được boot từ CDROM (/var/lib/libvirt/images/CentOS-7-x86_64-Minimal-1804.iso)

+ Card mạng: Sử dụng Linux Bridge br0
```

```sh
<domain type='kvm'>
  <name>anhnd-kvm01</name>
  <uuid>cf33d5ca-c477-11ec-b2b7-5254000352f7</uuid>
  <memory unit='KiB'>2097152</memory>
  <currentMemory unit='KiB'>2097152</currentMemory>
  <vcpu placement='static'>2</vcpu>
  <os>
    <type arch='x86_64' machine='pc-i440fx-rhel7.0.0'>hvm</type>
    <boot dev='cdrom'/>
  </os>
  <features>
    <acpi/>
    <apic/>
  </features>
  <cpu mode='custom' match='exact' check='partial'>
    <model fallback='allow'>SandyBridge</model>
  </cpu>
  <clock offset='utc'>
    <timer name='rtc' tickpolicy='catchup'/>
    <timer name='pit' tickpolicy='delay'/>
    <timer name='hpet' present='no'/>
  </clock>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>destroy</on_crash>
  <pm>
    <suspend-to-mem enabled='no'/>
    <suspend-to-disk enabled='no'/>
  </pm>
  <devices>
    <emulator>/usr/libexec/qemu-kvm</emulator>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/var/lib/libvirt/images/anhnd-kvm01.qcow2'/>
      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
    </disk>
    <disk type='file' device='cdrom'>
      <driver name='qemu' type='raw'/>
	  <source file="/var/lib/libvirt/images/CentOS-7-x86_64-Minimal-1804.iso"/>
      <target dev='hda' bus='ide'/>
      <readonly/>
      <address type='drive' controller='0' bus='0' target='0' unit='0'/>
    </disk>
    <controller type='usb' index='0' model='ich9-ehci1'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x7'/>
    </controller>
    <controller type='usb' index='0' model='ich9-uhci1'>
      <master startport='0'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0' multifunction='on'/>
    </controller>
    <controller type='usb' index='0' model='ich9-uhci2'>
      <master startport='2'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x1'/>
    </controller>
    <controller type='usb' index='0' model='ich9-uhci3'>
      <master startport='4'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x2'/>
    </controller>
    <controller type='pci' index='0' model='pci-root'/>
    <controller type='ide' index='0'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x1'/>
    </controller>
    <controller type='virtio-serial' index='0'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
    </controller>
    <interface type='bridge'>
      <mac address='4c:d9:8f:be:d7:8b'/>
      <source bridge='vlan16'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
    </interface>
    <serial type='pty'>
      <target type='isa-serial' port='0'>
        <model name='isa-serial'/>
      </target>
    </serial>
    <console type='pty'>
      <target type='serial' port='0'/>
    </console>
    <channel type='unix'>
      <target type='virtio' name='org.qemu.guest_agent.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='1'/>
    </channel>
    <channel type='spicevmc'>
      <target type='virtio' name='com.redhat.spice.0'/>
      <address type='virtio-serial' controller='0' bus='0' port='2'/>
    </channel>
    <input type='tablet' bus='usb'>
      <address type='usb' bus='0' port='1'/>
    </input>
    <input type='mouse' bus='ps2'/>
    <input type='keyboard' bus='ps2'/>
    <graphics type='spice' autoport='yes'>
      <listen type='address'/>
      <image compression='off'/>
    </graphics>
    <sound model='ich6'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
    </sound>
    <video>
      <model type='qxl' ram='65536' vram='65536' vgamem='16384' heads='1' primary='yes'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
    </video>
    <redirdev bus='usb' type='spicevmc'>
      <address type='usb' bus='0' port='2'/>
    </redirdev>
    <redirdev bus='usb' type='spicevmc'>
      <address type='usb' bus='0' port='3'/>
    </redirdev>
    <memballoon model='virtio'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x08' function='0x0'/>
    </memballoon>
  </devices>
</domain>
```
