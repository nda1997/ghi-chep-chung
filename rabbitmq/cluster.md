Node type
Một node trong RabbitMQ có thể là disk node hoặc RAM node. (Chú ý: disk và disc được sử dụng thay thế cho nhau, ví dụ trong cú pháp file cấu hình hoặc status messages thường sử dụng disc).

RAM nodes chỉ lưu các state của nó trong memory (đối với nội dung của queue cố định hoặc quá lớn để lưu trên memory thì có thể lưu trên disc).
Disk nodes giữ state trên cả memory và disk. Bởi vì RAM nodes không ghi nội dung lên disk như disk nodes, nên chúng hoạt động nhanh hơn. Tuy nhiên, cần lưu ý rằng do queue data luôn được lưu trên disc, nên phần cải thiện performance sẽ chỉ ảnh hưởng tới resources management (như adding/removing queues, exchanges, hoặc vhosts), chứ không cải thiện hiệu năng của publishing hay consuming.

Bởi vì state được replicate trên tất cả các node trong cluster, nên có thể chỉ cần có một disk node trong cluster để lưu state của cluster một cách an toàn. Tuy nhiên điều này không được khuyến nghị do tiềm ẩn nguy cơ mất mát dữ liệu.

Erlang cookie
Erlang nodes sử dụng cookie xác thực cho giao tiếp giữa các node. Cookie là một chuỗi ký tự các chữ cái với độ dài bất kỳ. Erlang tự động tạo file cookie ngẫu nhiên khi RabbitMQ server khởi động. 2 nodes có thể giao tiếp với nhau phải có cùng cookie, nên ta có thể copy file cookie từ một node sang các node còn lại trong cluster.
Trên Linux, file cookie đặt trong đường dẫn /var/lib/rabbitmq/.erlang.cookie hoặc $HOME/.erlang.cookie.

Mặc định các queue trong RabbitMQ cluster chỉ nằm trên một node duy nhất mà chúng được khai báo trước tiên. Không giống như exchange và binding luôn luôn nằm trên mọi node. Tuy nhiên, queue cũng có thể cấu hình để mirror trên nhiều node. Mỗi "mirrored queue" gồm 1 master và nhiều slave, và slave đầu tiên lên làm master mới nếu master cũ bị mất.

Message được publish tới queue sẽ được replicate đến tất cả các slave. Slave sẽ drop message mà nó nhận được. Do đó Queue mirroring chỉ nâng cao tính sẵn sàng chứ không phân bố tải trên các node.
