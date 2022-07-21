# note-aws-developer-associate-level

### SQS:
 - Thời gian lưu trữ message có thể setting: 1 - 14 days. Mặc định là 4 ngày
 
 - #### messageVisibleTimeout:
   Để tránh không cho nhiều consumer xử lý cùng 1 message thì có messageVisibleTimeout, có thể hiểu đây là thời gian tối đa consumer được phép xử lý message (default = 30s, minimum = 0, maximum = 12h)
 - #### delay queue:
   Để set thời gian delay cho message enqueue => sử dụng delayQueue (default = 0s, minimum = 0s, maximum = 15 minutes)
 => Sự khác biệt giữa messageVisibleTimeout và delayQueue là:
   - messageVisibleTimeout sẽ bị ẩn đối với các consumer khác MỖI LẦN có 1 consumer nhận message
   - delayQueue sẽ ẩn message đối với TẤT CẢ consumer lần đầu tiên nó được đưa vào queue

 - #### Size message:
   - Tối thiểu là 1 byte (1 ký tự)
   - Max size sẽ là 256KB
   - Có thể sử dụng Amazon SQS Extended Client Library for Java để tăng kích thước tối đa lên 2GB
 
### Elastic load balancer
 - Khi tạo load balancer => cần tạo target group => cần chỉ định target type => Khi tạo xong target group KHÔNG THỂ thay đổi target type
 - Có 3 loại target types:
   - instance: chỉ định bởi các ID instance
   - ip: Chỉ định bởi các IP (Có thể đặt range dải mạng, etc: 10.0.0.0/8)
   - Lambda: Mục tiêu là 1 hàm lambda

### Auto Scaling:
 - Health check + replace unhealth instance
 
### S3:
 - Đối với event notification, nếu không bật chế độ version thì khi 2 request đều ghi vào 1 đối tượng => có thể sẽ chỉ có 1 event notification được trigger
