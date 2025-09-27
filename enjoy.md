# cấu hình load balancing (nginx)

Trước khi đi vào cấu hình thì tôi muốn nói rằng nó chỉ là một cấu hình cơ bản nhất mà trong quá trình học tập tôi làm, tôi cũng tham khảo các
nguồn trên mạng && các mô hình AI nữa. Chính vì vậy nếu có gì cần chỉnh sửa và góp ý để nó hoàn thiện hơn thì mọi người cứ đóng góp nhiệt tình
nhé. Còn bây giờ thì bắt tay vào làm thôi.

Mô tả sơ qua những gì mà tôi sẽ cấu hình nhé:

- khi người dùng gửi request đến server thì thay vì gửi trực tiếp đến IP của server thì sẽ gửi đến 1 IP ảo, IP ảo này cũng là mấu chốt để liên 
kết các node lại với nhau để khi 1 node này chết thì sẽ chuyển tiếp request đến các node còn lại thông qua IP ảo đó chứ không bị lỗi hiển thị
với người dùng.


