# Load Balancing và Hight Availability (HA)

## Ở cấp độ cao:

`Load Balancing`: đảm bảo các yêu cầu từ client đến server được phân phối hiệu quả trên nhiều máy
chủ hoặc dịch vụ. Thay vì gây quá tải cho một máy chủ duy nhất, lưu lượng được phân bổ một cách 
thông minh, cải thiện hiệu suất và khả năng chịu lỗi.

`HA`: đảm bảo hệ thống của bạn vẫn có thể truy cập được ngay cả khi một số bộ phận của 
cơ sở hạ tầng gặp sự cố. Bằng cách tích hợp tính năng dự phòng và chuyển đổi dự phòn
vào thiết kế, HA bảo vệ hệ thống khỏi sự cố ngừng hoạt động và giảm thiểu thời giam chết.

## Đi sâu hơn nhé:

`Bộ cân bằng tải (LB)` hoạt động như thế nào

`Bộ cân bằng tải` hiện đại không chỉ xáo trộn các yêu cầu. Nó còn đóng nhiều vai trò quan trọng giúp ứng dụng phản hồi nhanh và ổn định.

- Đầu tiên, nó giám sát tình trạng hoạt động của các máy chủ back-end . Hãy tưởng tượng ba máy chủ nằm sau một bộ cân bằng tải, nếu một máy chủ gặp sự cố hoặc bắt đầu trả về lỗi, bộ cân bằng tải sẽ nhận ra và đơn giản là dừng gửi yêu cầu đến đó. Khi máy chủ phục hồi, nó có thể được đưa trở lại vòng quay, đôi khi từ từ để tránh bị quá tải lưu lượng ngay khi trực tuyến.

- Thứ hai, nó có thể quản lý các phiên người dùng . Nhiều ứng dụng cần theo dõi trạng thái của người dùng, chẳng hạn như các mặt hàng trong giỏ hàng. Nếu các phiên đó được lưu trữ trong bộ nhớ trên máy chủ, bộ cân bằng tải phải đảm bảo cùng một người dùng luôn truy cập vào cùng một máy chủ. Tính "dính" này rất tiện lợi, nhưng cũng có cái giá của nó: nó có thể gây mất cân bằng nếu quá nhiều người dùng bị ràng buộc vào cùng một máy.

- Thứ ba, bộ cân bằng tải thường xử lý mã hóa . Thay vì mỗi máy chủ tự giải mã lưu lượng HTTPS, bộ cân bằng tải có thể thực hiện việc này một lần tại biên. Điều này giúp giảm tải CPU và giúp việc quản lý chứng chỉ đơn giản hơn nhiều.

- Cuối cùng, bộ cân bằng tải tốt giúp quá trình chuyển đổi diễn ra mượt mà. Khi bạn đưa máy chủ ngoại tuyến để bảo trì, chúng có thể "rút" kết nối để người dùng đang hoạt động không bị thoát ra giữa chừng khi đang yêu cầu. Khi một máy chủ mới trực tuyến, chúng có thể đưa máy chủ vào hoạt động từ từ để tránh quá tải. Những chi tiết này rất quan trọng trong môi trường sản xuất, nơi sự khác biệt giữa việc chuyển đổi dự phòng trơn tru và sự gia tăng đột biến lỗi thường nằm ở các lựa chọn cấu hình nhỏ.

Sau đây là một ví dụ nhanh về HAProxy cho thấy một số tính năng này hoạt động như thế nào:

```
frontend web_front
    bind *:80
    default_backend web_servers
 
backend web_servers
    balance leastconn
    option httpchk GET /health
    server web1 10.0.0.11:80 check
    server web2 10.0.0.12:80 check
    server web3 10.0.0.13:80 check

```

Trong thiết lập này, HAProxy kiểm tra tình trạng hoạt động của từng máy chủ trước khi định tuyến yêu cầu. Nó luôn gửi lưu lượng mới đến máy chủ có ít kết nối hoạt động nhất, và nếu một máy chủ ngừng hoạt động, lưu lượng sẽ tự động chuyển sang các máy chủ khác mà không bị gián đoạn.

Các loại bộ cân bằng tải
Không phải tất cả bộ cân bằng tải đều hoạt động theo cùng một cách. Sự khác biệt thường nằm ở vị trí chúng hoạt động trong ngăn xếp mạng và loại quyết định chúng đưa ra về lưu lượng.

Ở mức đơn giản hơn là các bộ cân bằng tải Lớp 4 , hoạt động ở lớp vận chuyển của mô hình OSI. Chúng không quan tâm đến chi tiết của yêu cầu, mà chỉ quan tâm đến địa chỉ IP và các cổng TCP hoặc UDP liên quan. Vì không xem xét dữ liệu, chúng có thể chuyển tiếp lưu lượng rất nhanh chóng và hiệu quả. Điều này làm cho cân bằng tải Lớp 4 phù hợp với các khối lượng công việc lớn, độ phức tạp thấp, chẳng hạn như kết nối TCP thô hoặc các dịch vụ web đơn giản không cần định tuyến chi tiết.

Lên tầng trên, bộ cân bằng tải lớp 7 sẽ xem xét nội dung thực tế của các yêu cầu, chẳng hạn như tiêu đề HTTP, cookie, hoặc thậm chí là đường dẫn yêu cầu. Khả năng hiển thị bổ sung này cho phép đưa ra các quyết định thông minh hơn: ví dụ: định tuyến tất cả các yêu cầu hình ảnh đến một cụm được tối ưu hóa cho các tệp tĩnh, hoặc chuyển hướng khách hàng cao cấp đến một nhóm máy chủ chuyên dụng. Đánh đổi là việc kiểm tra các yêu cầu sẽ làm tăng thêm một chút chi phí xử lý, nhưng trên thực tế, cân bằng tải lớp 7 thường đáng giá nhờ tính linh hoạt mà nó mang lại.

Ngoài ra còn có các chiến lược nằm ngoài sự phân biệt truyền thống giữa L4/L7. Cân bằng tải dựa trên DNS phân tán lưu lượng bằng cách trả về các địa chỉ máy chủ khác nhau tùy thuộc vào phản hồi của DNS. Cách tiếp cận này đơn giản và phân tán trên toàn cầu, nhưng có một điểm cần lưu ý: bộ nhớ đệm DNS đồng nghĩa với việc chuyển đổi dự phòng có thể chậm hoặc không ổn định.

Đối với các tổ chức hoạt động trên quy mô toàn cầu, cân bằng tải toàn cầu trở nên thiết yếu. Người dùng ở Paris không nên yêu cầu của họ được xử lý từ một trung tâm dữ liệu ở California nếu đã có một trung tâm ở Frankfurt sẵn sàng đáp ứng. Các hệ thống toàn cầu sử dụng các kỹ thuật như định tuyến Anycast hoặc các dịch vụ đám mây được quản lý (chẳng hạn như định tuyến dựa trên độ trễ AWS Route 53 hoặc Google Cloud Load Balancing) để hướng người dùng đến khu vực hoạt động tốt gần nhất.

Điểm mấu chốt là mỗi loại bộ cân bằng tải đều phục vụ một mục đích riêng. Một ứng dụng web có thể phụ thuộc vào nhiều loại cùng lúc: DNS để gửi người dùng đến vùng gần nhất, bộ cân bằng tải L7 để chuyển hướng yêu cầu của họ đến đúng dịch vụ, và một nền tảng như Cycle.io hoặc bộ cân bằng tải đám mây được quản lý để duy trì lưu lượng truy cập ổn định trong môi trường. Việc lựa chọn kết hợp phù hợp phụ thuộc vào quy mô hệ thống, độ phức tạp của lưu lượng truy cập và mức độ kiểm soát bạn cần.


Hiểu về `tính khả dụng cao (HA)`

`Tính khả dụng cao (HA)` là gì?

Nếu cân bằng tải là về việc phân phối lưu lượng, thì tính khả dụng cao là về việc đảm bảo dịch vụ vẫn hoạt động khi có sự cố. Trên thực tế, điều đó có nghĩa là thiết kế hệ thống sao cho chỉ một lỗi nhỏ - dù là máy chủ hỏng, đường truyền mạng bị lỗi hay một thành phần hoạt động không bình thường - cũng không khiến mọi thứ ngừng hoạt động.

Tính khả dụng cao, thường được viết tắt là HA , thường được thể hiện bằng thời gian hoạt động. Một hệ thống có độ khả dụng "bốn số chín" (99,99%) được phép ngừng hoạt động dưới một giờ mỗi năm. Để đạt được điều này, các kỹ sư tích hợp tính năng dự phòng vào mọi lớp của ngăn xếp. Thay vì dựa vào một cơ sở dữ liệu, có thể có một cơ sở dữ liệu chính và một cơ sở dữ liệu dự phòng sẵn sàng tiếp quản. Thay vì một trung tâm dữ liệu, các khối lượng công việc có thể chạy trong hai hoặc ba trung tâm, mỗi trung tâm có thể xử lý lưu lượng nếu trung tâm khác gặp sự cố.

Điều quan trọng là phải phân biệt HA với phục hồi sau thảm họa . Tính khả dụng cao (HA) là việc duy trì các dịch vụ trực tuyến khi đối mặt với các sự cố dự kiến , chẳng hạn như máy chủ bị sập hoặc tủ rack mất điện. Phục hồi sau thảm họa là những gì xảy ra trong trường hợp xấu nhất: mất điện toàn bộ khu vực, hỏng dữ liệu nghiêm trọng hoặc thiên tai. Cả hai đều là một phần của khả năng phục hồi, nhưng HA là tuyến phòng thủ đầu tiên giúp giảm thiểu thời gian ngừng hoạt động mà người dùng có thể nhìn thấy hàng ngày.

Bạn sẽ thấy tính khả dụng cao ở mọi nơi, độ tin cậy là yếu tố then chốt. Các nền tảng giao dịch tài chính không thể ngừng hoạt động dù chỉ vài phút. Hệ thống chăm sóc sức khỏe phải hoạt động trực tuyến 24/7 để bác sĩ có thể truy cập dữ liệu bệnh nhân trong trường hợp khẩn cấp. Ngay cả các ứng dụng hướng đến người dùng, như mạng xã hội hoặc nền tảng phát trực tuyến, cũng phụ thuộc rất nhiều vào HA để người dùng không bao giờ phải đắn đo khi nhấp vào nút phát hoặc làm mới nguồn cấp dữ liệu.


Mối quan hệ giữa `cân bằng tải` và `tính khả dụng cao`

Họ bổ sung cho nhau như thế nào

Cân bằng tải và tính khả dụng cao thường được xem là hai khái niệm riêng biệt, nhưng trên thực tế, chúng là hai mặt của một vấn đề. Tính khả dụng cao đảm bảo hệ thống của bạn được xây dựng để chống chịu được các sự cố, còn cân bằng tải đảm bảo lưu lượng thực sự được truyền tải qua các sự cố đó theo thời gian thực.

Hãy tưởng tượng một cụm máy chủ ứng dụng chạy trong hai vùng khả dụng. Tính dự phòng vẫn tồn tại: nếu một vùng ngừng hoạt động, vùng còn lại vẫn có thể xử lý lưu lượng. Nhưng nếu không có bộ cân bằng tải ở phía trước, các yêu cầu của người dùng vẫn sẽ cố gắng tiếp cận vùng bị lỗi và họ sẽ gặp lỗi cho đến khi bản ghi DNS hết hạn hoặc ai đó cập nhật cấu hình thủ công. Tính dự phòng vẫn tồn tại, nhưng nó không được sử dụng hiệu quả.

Mặt khác, một bộ cân bằng tải không có tính sẵn sàng cao hỗ trợ chỉ có thể làm được đến mức đó. Nếu tất cả máy chủ đều trỏ đến một cơ sở dữ liệu duy nhất và cơ sở dữ liệu đó bị lỗi, bộ cân bằng tải sẽ không có nơi nào đáng tin cậy để gửi lưu lượng. Trong trường hợp đó, bộ cân bằng tải đang phân phối lỗi thay vì ngăn chặn nó.

Sức mạnh thực sự đến khi cả hai cùng hoạt động. Bộ cân bằng tải phát hiện lỗi và định tuyến lại lưu lượng, trong khi thiết kế khả dụng cao đảm bảo luôn có nơi nào đó an toàn để gửi dữ liệu. Cùng nhau, chúng tạo ra những hệ thống không chỉ tồn tại sau sự cố mà còn giảm thiểu tác động đến trải nghiệm người dùng.

Một ví dụ phổ biến là trong kiến ​​trúc vi dịch vụ. Cổng API front-end có thể sử dụng cân bằng tải để phân bổ các yêu cầu trên nhiều phiên bản dịch vụ. Về cơ bản, các dịch vụ này được triển khai trên nhiều vùng với cơ chế dự phòng. Nếu một vùng bị lỗi, bộ cân bằng tải sẽ tự động chuyển lưu lượng sang vùng hoạt động bình thường, trong khi thiết kế HA đảm bảo các phiên bản dịch vụ đã chạy và sẵn sàng tiếp nhận thêm tải. Người dùng sẽ không bao giờ nhận thấy sự gián đoạn.

Sự cộng hưởng này là lý do tại sao các cơ sở hạ tầng trưởng thành hiếm khi triển khai cái này mà không có cái kia. Cân bằng tải và tính khả dụng cao bổ trợ cho nhau chặt chẽ đến mức chúng được coi là một giải pháp trọn gói.
