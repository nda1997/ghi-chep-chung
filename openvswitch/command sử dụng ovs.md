# Command sử dụng Openvswitch

### ovs-vsctl

`show` : hiển thị các bridge hiện có và các port kết nối 

![image](https://user-images.githubusercontent.com/50499526/168210333-c250549c-b76a-4223-8aa7-14abce36e3c8.png)

`add-br <tên bridge > ` : tạo 1 bridge mới
`del-br <tên bridge > ` : xóa bridge và toàn bộ các port được gắn vào nó
`add-br < tên >  < parent bridge >  vlan ` : tạo 1 fake bridge và gán vào 1 bridge khác có gán tag vlan

![image](https://user-images.githubusercontent.com/50499526/168210706-d79b09c9-d7fc-4e9e-8756-e5b800a99bd8.png)

` add-port < bridge > <port muốn gán > `: thêm port vào bridge

` ovs-vsctl \
    -- add-port br0 patch0 \
    -- set interface patch0 type=patch options:peer=patch1 \
    -- add-port br1 patch1 \
    -- set interface patch1 type=patch options:peer=patch0 ` : kết nối trunking giữa 2 bridge
    
` set-controller < brigde > tcp:<controller_ip>:<port> ` : kết nối bridge tới 1 controller xác định

**note** : controller có thể thu thập và phát hành , quản lý thông tin liên quan đến bridge 
    + set-controller br0 tcp:192.168.1.2:8080 
    + sau đó ta có thể truy cập GUI web tại địa chỉ  ` http://182.168.1.2:8080/ui/index.html `

### ovs-appctl 
` ofproto/trace <bridge >  in_port=1 ` : theo dõi luồng đi 

![image](https://user-images.githubusercontent.com/50499526/168212448-7da21728-1606-4ebf-8c4f-7b362c6f67cb.png)

`vlog/list ` : liệt kê danh sách mô đun log và cấp độ sử dụng chúng

![image](https://user-images.githubusercontent.com/50499526/168225547-fb567b5d-4116-4a2d-b7a8-e2ab941a6034.png)

### ovs-ofctl

`show <bridge > ` : hiện thông tin về bridge , bảng flow và cả các port kết nối đến 

![image](https://user-images.githubusercontent.com/50499526/168226501-c6e959a4-5766-43a3-943c-9646f09f898c.png)

`dump-tables <bridge > ` : in ra bảng flow đang sử dụng 

![image](https://user-images.githubusercontent.com/50499526/168226618-78c97003-0b60-4776-b29d-7fd0e93f21d3.png)

`dump-flows < bridge > ` :  hiện thông tin về các flow entry hiện hữu 
` add-flow <bridge >  "table=1,priority=1000,actions=normal" ` : thêm 1 flow vào bridge với các tham số . *action*  có thể để mặc định hoặc cấu hình chuyển hướng khác nhau ( output , resubmit ...)
