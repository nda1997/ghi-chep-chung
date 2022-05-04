# Backup and Restore KVM
- Có rất nhiều cách để backup/restore VM trong KVM . ví dụ như sử dụng script , phần mềm bên thứ 3 ,...
- Dưới đây sử dụng cách backup bằng tay
## 1.1 Backup a VMs in kvm
 
- Đầu tiên ta kiểm tra xem có những VM nào đang chạy

``` virsh list --all ```

![image](https://user-images.githubusercontent.com/50499526/166639003-57ec6d70-ff41-4591-af9d-62f63f7fd944.png)

Như trên ví dụ ta muốn backup Vm anhnd-node1 . Tiến hành shutdown VM

``` virsh shutdown tên_VM ```

- Thông thường 1 máy ảo sẽ được cấu tạo từ 2 thành phần là file XML và ổ đĩa ảo chứa dữ liệu
    + ta copy file xml sang thư mục backup 
    
    ``` virsh dumpxml tên_VM > /opt/backupvm/tên_VM.xml
    
    + Xác định vị trí ổ đĩa của VM
    
    ``` virsh domblklist tên_VM ```
    
![image](https://user-images.githubusercontent.com/50499526/166639877-a2694ac1-2c6e-4fae-8964-6957c5ab0cee.png)
    
Ở đây đường dẫn là /kvm/anhnd-node1.qcow2

- Ta copy file disk sang thư mục backup

``` cp /kvm/tên_VM.qcow2 /opt/backupvm ```

![image](https://user-images.githubusercontent.com/50499526/166640277-2b869b89-654f-4fc4-9cb0-6001ecdf1a37.png)

## 1.2 Restore a VMs

- Undefine VM ( quá trình này sẽ xóa file xml của VM)
``` virsh undefine tên_VM ```

- Xóa bỏ ổ đĩa của VM 

``` rm -rf /path-to-harddrive ```

![image](https://user-images.githubusercontent.com/50499526/166641313-b3811b18-52f6-4dc2-8eec-983c95e2d6ec.png)

- Khôi phục lại VM : Quá trình này đơn giản là ta copy lại file xml , ổ đĩa của VM về thư mục chính của KVM và tiến hành define lại nó

![image](https://user-images.githubusercontent.com/50499526/166641709-45d436d2-f226-4986-8b62-ee5c1f0ba388.png)

``` virsh define --file /path-to-file-xml ```

![image](https://user-images.githubusercontent.com/50499526/166641934-4747f0ea-540d-4343-95cf-e8dfcc787702.png)

- Như vậy là quá trình backup đã thành công 

