# Báo cáo cá nhân — Day 01: Lọc tin nhắn rác bằng ML, DL, Embedding và LLM

> Đổi tên file này thành `MSSV_HoTen.md` của bạn, ví dụ `2A20260000_NguyenVanAn.md`, rồi điền từng mục. Xoá các dòng hướng dẫn in nghiêng khi xong.

- **MSSV:** 2A202603483
- **Họ và tên:** ĐẶNG LONG NHẬT
- **SEED** (in ở ô đầu notebook): 5527

## 1. Bằng chứng

| Tệp | Có? |
| --- | --- |
| [`day01_lab.ipynb`](day01_lab.ipynb) — notebook đã chạy, còn output | Y |
| [`lab1_report.json`](lab1_report.json) — do ô "Báo cáo cuối" sinh ra | Y |
| [`hand_calc.jpg`](hand_calc.jpg) — ảnh tính tay có các bước | Y |

## 2. Kết quả chính

                                F1 test thường (SUB)  F1 50 case khó  \
1. BoW + LogReg                                0.857           0.333   
2. BoW + MLP                                   0.947           0.462   
3a. Emb + kNN                                  0.974           0.764   
3b. Emb + LogReg                               0.974           0.640   
3c. Emb + MLP head                             0.974           0.667   
4a. LLM nhỏ zero-shot                          0.000           0.000   
4d. LLM nhỏ + prompt phòng thủ                 0.000           0.000   

                                acc 50 case khó  
1. BoW + LogReg                            0.52  
2. BoW + MLP                               0.58  
3a. Emb + kNN                              0.74  
3b. Emb + LogReg                           0.64  
3c. Emb + MLP head                         0.66  
4a. LLM nhỏ zero-shot                      0.42  
4d. LLM nhỏ + prompt phòng thủ             0.42  

## 3. Dự đoán và thực tế

| Dự đoán | Tôi đoán gì? | Kết quả thực tế | Vì sao lệch? |
|---|---|---|---|
| **`llm_models`** | Model lớn đắt hơn khoảng **1,5 lần**. | Chi phí ước tính **0,00316 USD / 0,00032 USD ≈ 9,9 lần**. | Tôi chưa tính theo đơn giá token và lượng token thực tế của từng model; không thể suy tỷ lệ chi phí chỉ từ kích thước model. |
| **`dl_relu`** | Bỏ ReLU thì F1 gần LogReg Phần 1, khoảng **0.880**. | Mạng không ReLU đạt **F1 = 0.940**, cao hơn **0.060**. | Hai lớp Linear tương đương một lớp về khả năng biểu diễn, nhưng cách huấn luyện khác nhau: mạng dùng Adam và mini-batch, còn LogReg dùng gradient descent toàn bộ dữ liệu, nên kết quả tối ưu có thể khác. |
| **`hard_eval`** | Accuracy của BoW+LogReg khoảng **55–70%**, BoW+MLP khoảng **60–75%**. | Accuracy lần lượt chỉ **52%** và **58%**. | Tôi đánh giá quá cao khả năng tổng quát hóa. Bộ test khó chứa spam né từ khóa và viết như hội thoại, trong khi BoW phụ thuộc nhiều vào các từ đã gặp khi train. |
## 4. Phân tích 50 case khó


BoW + LogReg tụt nhiều nhất: F1 từ 0.857 trên SUB 40 xuống 0.333 trên 50 case khó, giảm khoảng 0.524. Model gặp khó nhất ở nhóm spam giọng tự nhiên, chỉ đúng 1/10 case (10%); tiếp theo là thao túng/lừa đảo (12,5%) và né từ khóa (16,7%).
| Case | Nội dung tiêu biểu | Cơ chế khiến model sai |
|---|---|---|
| **#12 – Spam giọng tự nhiên** | “my friend gave me your number… looking for someone to chat with… check my profile at meetlocal.club” | Tin nhắn dùng giọng lịch sự, giả làm người quen và ít từ khóa spam nổi bật. BoW dựa vào các từ riêng lẻ nên khó nhận ra ý định dụ người nhận truy cập trang hẹn hò. |
| **#14 – Spam giọng tự nhiên** | “working from home and making about £300 a day… no experience needed… just text me back” | Nội dung được viết như lời chia sẻ cá nhân. BoW khó kết hợp các dấu hiệu “thu nhập cao, không cần kinh nghiệm, yêu cầu liên hệ” thành dấu hiệu lừa việc làm, nên bỏ sót spam. |

## 5. Thử thách cá nhân

*Đề theo SEED của bạn, cách làm, kết quả, giải thích.*

## 6. Phương án cho sản phẩm thật

3c. Encoder MiniLM + MLP head làm phương án chính: F1 test = 0.959, F1 SUB 40 = 0.974, p95 = 22.068 ms/tin, đáp ứng giới hạn 50 ms. Throughput đo được 153.868 tin/giây, cao hơn mức trung bình cần thiết khoảng 116 tin/giây cho 10 triệu SMS/ngày.

Đánh đổi là tốn tài nguyên và suy luận chậm hơn BoW, nhưng chất lượng tốt hơn. Chi phí API bằng 0 trong bảng vẫn chưa tính chi phí máy chủ. Ngoài ra, F1 trên 50 case khó chỉ đạt 0.667, nên cần bổ sung dữ liệu spam mới trước khi triển khai rộng.

## 7. AI log và điều đã học

| Phần | Bạn hỏi AI gì (prompt) | AI trả lời/viết gì | Bạn đã sửa / kiểm chứng gì |
|---|---|---|---|
| Phần 3-4 | Công thức toán học tổng quát (3), hỗ trợ viết test case, kết luận và check code huấn luyện (4)| AI bổ sung code, giải thích và case tham khảo | Sửa code AI feedback, chuẩn hóa lại kiểu dữ liệu, kiếm chứng kết quả test |
