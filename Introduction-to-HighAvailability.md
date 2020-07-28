![](https://i.imgur.com/Smcge1h.png)

# Tổng quan về High Availability - Tính sẵn sàng cao.

## Giới thiệu

\Nhu cầu về thiết kế hệ thống đáng tin cậy và hiệu năng cao để phục vụ cho các hệ thống quan trọng ngày một tăng, các điều khoản về khả năng mở rộng và tính sẵn sàng cao lại càng phổ biến. Khi mà việc xử lý tăng khả năng chịu tải của hệ thống làm mối quan tâm chung, giảm downtime(thời gian hệ thống không hoạt động) và loại bỏ các [single point of failure](https://en.wikipedia.org/wiki/Single_point_of_failure) lại càng quan trọng. High availability là một chất lượng cho thiết kế hệ thống có quy mô mà giải quyết các cân nhắc sau này.

## High Availability là gì?

Trong khoa học máy tính, *Availability* được dùng để mô tả khoảng thời gian mà dịch vụ sẵn sàng, cũng như là thời gian yêu cầu bởi hệ thống để có thể phản hồi yêu cầu được tạo bởi người dùng. High availability là một chất lượng của một hệ thống hay một thành phần mà đảm bảo hiệu xuất hoạt động cao trong một khoảng thời gian nhất định.

### Đo lượng tính sẵn sàng

Tính sẵn sàng(availibility) thường được thể hiện bằng tỷ lệ phần trăm chỉ ra uptime(thời gian hoạt động) mong muốn của một hệ thống hay một thành phần trong một khoảng thời gian nhất định, nếu là 100% thì là hệ thống không bao giờ lỗi và luôn luôn hoạt động. Ví dụ, hệ thống cam kết tính sẵn sàng đảm bảo 99% trong một năm, tức là hệ thống có thể có downtime tối đa 3.65 ngày(1%).

Các giá trị này được tính toán dựa trên một số yếu tố, bao gồm cả thời gian bảo trì theo lịch trình và theo lịch trình, cũng như thời gian để phục hồi sau sự cố hệ thống có thể xảy ra.

## High Availability hoạt động như thế nào.

Chức năng sẵn sàng cao như là một cơ chế phản hồi các thất bại cho hạ tầng. Cách nó hoạt động thì khá cơ bản về mặt khái niệm tuy nhiên nó thường yêu cầu các phần mềm và các cấu hình chuyên dụng.

## Cần High Availability khi nào?

Khi thiết lập một hệ thống production mạnh mẽ, tối thiểu downtime và việc gián đoạn dịch vụ được ưu tiên cao. Bất kể độ tin cậy của hệ thống hay phần mềm của bạn như thế nào, các vấn đề vần có thể sảy ra mà khiến cho phần mềm của bạn trên server của bạn bị sập.

Việc triển khai tính sẵn sàng cao(HA) cho hạ tầng cảu bạn là chiến lược hữu ích để giảm ảnh hưởng của các sự kiện loại này. Hệ thống có tính sẵn sàng cao có thể tự động khôi phục khi hệ thống sảy ra lỗi.

## Hệ thống như nào thì được coi là hệ thống có tính sẵn sàng cao.

Một trong như mục tiêu của tính sẵn sàng cao là loại bỏ các single point of failure trong hạ tầng của bạn. Single point of failure(SPOF) lf một thành phần trong technology stack(tập hợp các thành phần,công nghệ tạo thành nền tảng để chạy dịch vụ của bạn) mà có thể khiến cho cả dịch vụ bị gián đoạn nếu nó không sẵn sàng. Như vậy, bất kì thành phần nào là điều kiện tiên quyết cho một chức năng thích hợp của ứng dụng mà không có dự phòng thì bị coi là một SPOF.

Để loại bỏ SPOF, mỗi layer trong stack của bạn phải chuẩn bị dự phòng. Ví dụ, tưởng tượng bạn có một hạ tầng chứa hai web server giống hệt nhau và dự phòng cho nhau đứng sau một bộ cân bằng tải. Các yêu cầu đến dịch vụ sẽ được phân bổ đều trên hai webserver nhờ bộ cân bằng tải. Khi một webserver của bạn sập, bộ cân bằng tải sẽ chuyển hướng tất cả yêu cầu đến web server còn đang chạy.

Với hệ thống mô tả ở trên, Web server layer sẽ không bị coi là SPOF nữa vì nó có các thành phần phục vụ cùng nhiệm vụ để dự phòng và có bộ cân bằng tải để phát hiện lỗi và kịp thời thay đổi để khôi phục hệ thống.

Tuy nhiên, bộ cân bằng tải(Load balencer) ở trong mô hình này lại trở thành single point of failure. Vì nó liên quan đến việc định hướng cho các yêu cầu đến web server layer, mà lại không có dự phòng. Nếu nó sập, dù web server vẫn hoạt động nhưng yêu cầu lại không gửi đến được webserver, dẫn đến toàn hệ thống bị gián đoạn.

![](https://i.imgur.com/co7LdFw.png)

Bạn cũng có thể triển khai một bộ cân bằng tải nữa để làm dự phòng. Tuy nhiên nếu chỉ có dự phòng thì không đủ tính sẵn sàng cao. Một cơ chế phát hiện lỗi và thực hiện các hành động giúp hệ thống phục hồi. Không thể nào lại thêm một layer trên layer cân bằng tải để phát hiện lỗi và phục hồi cho layer bộ cân bằng tải, vì nếu như vậy thì layer mới lại trở thành một single point of failure của hệ thống.

Trong trường hợp này, một tiếp cận phân tán là cần thiết. Nhiều node dự phòng phải kết nối với nhau như mà một cụm(cluster) mà trong đó mỗi node đều có khả năng phát hiện lỗi và khôi phục như nhau.

![](https://i.imgur.com/Wi4kkUr.png)


Lúc này lại sinh ra một vấn đề khác là khi mà bộ cân bằng tải đang sử dụng sập và bộ cân bằng tải dự phòng hoạt động để thay thế thì một thay đổi DNS bắt buộc phải thực hiện. Thay đổi DNS để khi mà có yêu cầu gửi đến domain name thì yêu cầu sẽ được gửi đến bộ cân bằng tải dự phòng. Tuy nhiên một thay đổi như thế thối rất nhiều thời gian, dẫn đến downtime lớn cho hệ thống.

Một giải pháp mạnh mẽ và đáng tin cậy để giải quyết vấn đề này là sử dụng hệ thống cho phép việc ánh xạ địa chỉ IP linh hoạt, ví dụ như floating IPs. Việc ánh xạ địa chỉ Ip theo yêu cầu giúp loại bỏ các vấn đề về việc thay đổi DNS vì bản ghi DNS có thể chỏ đến một địa chỉ IP tĩnh và địa chỉ này lại có thể dễ dàng được ánh xạ lại sang node khác khi cần. DNS thì trỏ đến một địa chỉ IP trong khi địa chỉ IP lại có thể di chuyển giữa các server.


Ảnh động sau mô tả hạ tầng có tính sẵn sàng cao sử dụng Floating IPs.

![](https://i.imgur.com/eLkZ1N7.gif)



## Các thành phần hệ thống nào được yêu cầu cho tính sẵn sàng cao - High Availability.

Có một số thành phần phải được chuẩn bị kỹ lưỡng trước khi triển khai tính sẵn sàng cao trong thực tế. Hơn việc triển khai phần mềm :), tính sẵn sàng cao phụ thược vào các yếu tố như:
- **Môi trường**: Nếu tất cả server được đặt chung một vị trí địa lý nào đó, các điều kiện về mặt môi trường như là động đất, sóng thần có thể sẽ khiến cả hệ thống của bạn sập. Việc có các server dự phòng ở các vị trí địa lý khác nhau sẽ tăng độ tin cậy.
- **Phần cứng**: server có tính khả dụng cao còn nên có khả năng phục hồi khi mất điện và các lỗi phần cứng như ổ đĩa và network interfac.
- **Phần mềm**: Tất cả stack phần mềm, bao gồm hệ điều hành và cả ứng dụng nên chuẩn bị cho trường hợp gặp lỗi để phục hồi kịp thời.
- **Dữ liệu**: mất dữ liệu hoặc không nhất quán dữ liệu có thể được gây ra bởi nhiều kkhông chỉ hạn chế nguyên nhân do hard disk lỗi. Hệ thống có tính sẵn sàng cao phải tính toán đến việc bảo vệ dữ liệu khi sảy ra lỗi.
- **Mạng**: yếu tố về hệ thống mạng rất quan trọng. Một chiến lược mạng dự phòng phải được triển khai để có thể xử lý các sự cố.

## Ứng dụng nào sử dụng để cấu hình tính sẵn sàng cao?

Ở các tầng khác nhau trong hạ tầng có tính sẵn sàng cao lại có các điều kiện về phần mềm và phần cứng khác nhau. Tuy nhiên, với tầng ứng dụng, bộ cân bằng tải(load balancer - LB) là thành phần thiết yếu để tạo nên một ứng dụng có tính sẵn sàng cao.

**HAProxy** là một lựa chọn phổ biến cho việc cân bằng tải, vì nó có thể cân bằng tải giữa trên nhiều layer, nhiều loại server khác nhau, bao gồm cả database server.

Như đã nói ở trên, để loại bỏ single point of failure cần triển khai load balencer theo cụm và đằng sau một Floating IP. **Corosync** và **Pacemaker** phổ biến để triển khai những thiết lập như thế.



## Chú thích
- Đây là bài dịch có nguồn từ bài viết [What is High Availability?](https://www.digitalocean.com/community/tutorials/what-is-high-availability)
