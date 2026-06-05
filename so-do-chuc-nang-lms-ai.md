# Sơ đồ chức năng hệ thống LMS AI

## 1. Sơ đồ kiến trúc tổng thể

```mermaid
flowchart TB
  U["Người dùng<br/>Học viên / Giảng viên / Quản trị viên"] --> FE["Web/App<br/>Đăng nhập, học tập, kiểm tra,<br/>AI Assistant, quản lý nội dung"]
  FE --> GW["API Gateway<br/>Điều hướng request, xác thực,<br/>phân quyền, ghi log, chuẩn hóa dữ liệu"]

  GW --> AUTH["Auth Service<br/>Đăng nhập, phân quyền,<br/>quản lý phiên"]
  GW --> CORE["LMS Core<br/>Modular Monolith"]
  GW --> AI["AI Services<br/>Microservices"]

  CORE --> RDB["Relational Database<br/>Dữ liệu nghiệp vụ lõi"]
  CORE --> NOSQL["NoSQL Database<br/>Log, trạng thái, dữ liệu linh hoạt"]

  AI --> VDB["Vector Database<br/>Tri thức, tài liệu, embedding"]
  AI --> LLM["LLM API / Local LLM<br/>RAG, phân tích, sinh nội dung"]

  CORE <--> AI_SYNC["Đồng bộ chỉ số học tập,<br/>risk score, nhóm sinh viên,<br/>gợi ý can thiệp"]
  AI <--> AI_SYNC

  CORE --> NOTI["Hệ thống thông báo tự động"]
  CORE --> ONE["Cổng dịch vụ 1 cửa"]
  CORE --> EXAM["Hệ thống Khảo thí chuyên biệt"]
  CORE --> LARK["Lark/Base<br/>Group hỗ trợ, bàn giao danh sách"]
```

## 2. Sơ đồ cây chức năng chi tiết

```mermaid
mindmap
  root((LMS AI))
    Tầng giao diện & điều hướng
      Web/App
        Đăng nhập
        Học khóa học
        Làm bài kiểm tra
        Tương tác AI Assistant
        Quản lý nội dung
      API Gateway
        Điều hướng request
        Xác thực với Auth Service
        Phân quyền
        Ghi log
        Chuẩn hóa dữ liệu
    LMS Core - Modular Monolith
      Hệ đào tạo
        Chuyên ngành đào tạo
        Kỳ học
      Môn học
      Lớp học
        Điểm danh
        Thời khóa biểu
        Quản lý nhóm
        CSSV
        Kiểm tra đầu giờ
        Tài nguyên sau buổi học
        Đơn xin nghỉ phép
        Kiểm tra BTVN
      Khảo thí
        Kết nối hệ thống khảo thí
        Thống kê điểm thi
      E-learning
        Tải tài liệu học tập
      Thông báo tự động
      Quản lý dự án
        Tích hợp chỉ số AI
      Cổng dịch vụ 1 cửa
    AI Services - Microservices
      AI chấm BTVN / Project
      AI phân tích chỉ số học tập
        RAG
        Local LLM thử nghiệm
        Phân tích theo từng buổi
      AI Risk Score realtime
        Giảm tương tác LMS
        Nghỉ học tăng
        Tụt điểm nhanh
        Dynamic Re-Grouping
      AI Assistant
        Tra cứu hồ sơ
        Tóm tắt lịch sử học tập
        Tóm tắt hành vi
        Sinh kịch bản tư vấn
        Sinh nội dung nhắc nhở
        Gợi ý bài tập phụ đạo
      AI báo cáo & dự đoán
        Dashboard quản trị
        Báo cáo định kỳ
        Dự đoán fail môn
        Dự đoán bảo lưu
        Dự đoán PLO/CLO
      AI đánh giá giảng dạy
```

## 3. Sơ đồ triển khai theo giai đoạn

```mermaid
flowchart LR
  P1["Giai đoạn 1<br/>Nền tảng & nghiệp vụ lõi"] --> P2["Giai đoạn 2<br/>Tự động hóa CSSV"]
  P2 --> P3["Giai đoạn 3<br/>Tích hợp AI Services"]

  P1 --> P1A["API Gateway"]
  P1 --> P1B["Auth Service"]
  P1 --> P1C["Relational DB / NoSQL"]
  P1 --> P1D["Web/App cơ bản"]
  P1 --> P1E["Hệ đào tạo, môn học, lớp học"]
  P1 --> P1F["Điểm danh, TKB, quản lý nhóm"]
  P1 --> P1G["Cổng dịch vụ 1 cửa"]
  P1 --> P1H["Thông báo tự động"]

  P2 --> P2A["CSSV theo ngày"]
  P2A --> P2A1["Quét điểm danh hàng ngày"]
  P2A --> P2A2["Lọc sinh viên nghỉ không phép"]
  P2A --> P2A3["Cập nhật trạng thái nhanh"]
  P2A --> P2A4["Đồng bộ CTSV - Giảng viên"]

  P2 --> P2B["CSSV theo phân loại nhóm"]
  P2B --> P2B1["Tính R-point"]
  P2B --> P2B2["Kết hợp điểm môn trước"]
  P2B --> P2B3["Phân nhóm A/B/C/D"]
  P2B --> P2B4["Bàn giao danh sách"]
  P2B --> P2B5["Tạo group Lark/Base"]

  P3 --> P3A["AI Microservice"]
  P3A --> P3A1["Kết nối Vector DB"]
  P3A --> P3A2["Kết nối LLM API"]
  P3 --> P3B["RAG / Local LLM"]
  P3 --> P3C["Risk Score realtime"]
  P3 --> P3D["Dynamic Re-Grouping"]
  P3 --> P3E["AI Assistant"]
  P3 --> P3F["AI báo cáo & dự đoán"]
  P3 --> P3G["AI chấm BTVN / Project"]
  P3 --> P3H["AI đánh giá giảng dạy"]
```

## 4. Sơ đồ luồng CSSV và phân loại nhóm

```mermaid
flowchart TD
  START["Kết thúc buổi học / kết thúc môn"] --> DAILY{"Xử lý theo ngày?"}

  DAILY -->|Có| ATT["Quét dữ liệu điểm danh"]
  ATT --> ABS["Lọc sinh viên nghỉ học không phép"]
  ABS --> STATUS["Cập nhật trạng thái nhanh"]
  STATUS --> SYNC["Đồng bộ CTSV - Giảng viên"]
  SYNC --> NOTIFY["Gửi thông báo / nhắc nhở"]

  DAILY -->|Khi có điểm cuối môn| SCORE["Thu thập R-point và điểm môn trước"]
  SCORE --> CLASSIFY{"Phân loại sinh viên"}

  CLASSIFY --> A["Nhóm A<br/>Ý thức tốt<br/>Học lực khá/giỏi"]
  CLASSIFY --> B["Nhóm B<br/>Ý thức kém<br/>Học lực khá/giỏi"]
  CLASSIFY --> C["Nhóm C<br/>Ý thức tốt<br/>Học lực TB/yếu"]
  CLASSIFY --> D["Nhóm D<br/>Ý thức kém<br/>Học lực TB/yếu"]

  A --> GV1["Giảng viên phụ trách"]
  GV1 --> A_ACT["Tạo group Lark/Base<br/>Khen thưởng, thúc đẩy phát triển"]

  B --> CTSV1["Phòng CTSV phụ trách"]
  CTSV1 --> B_ACT["Nhắc nhở quy chế<br/>Cải thiện ý thức kỷ luật"]

  C --> GV2["Giảng viên phụ trách"]
  GV2 --> C_ACT["Cố vấn 1-1<br/>Gợi ý bài tập bổ sung<br/>Phụ đạo riêng"]

  D --> CTSV2["Phòng CTSV phụ trách"]
  CTSV2 --> D_ACT["Can thiệp sâu<br/>Phối hợp phụ huynh<br/>Xử lý rủi ro sớm"]

  A_ACT --> GROUP["Tự động khởi tạo group hỗ trợ"]
  B_ACT --> GROUP
  C_ACT --> GROUP
  D_ACT --> GROUP

  GROUP --> TRACK["Theo dõi kết quả can thiệp"]
  TRACK --> AI_RISK["AI Risk Score realtime<br/>Cập nhật nguy cơ và đề xuất chuyển nhóm"]
  AI_RISK --> REGROUP{"Dynamic Re-Grouping?"}
  REGROUP -->|Có| CLASSIFY
  REGROUP -->|Không| REPORT["Báo cáo CSSV / Dashboard quản trị"]
```

## 5. Sơ đồ chức năng AI Services

```mermaid
flowchart TB
  DATA["Dữ liệu đầu vào<br/>Điểm danh, điểm thi, tương tác LMS,<br/>BTVN, project, tài liệu học tập, hồ sơ sinh viên"] --> PIPE["AI Data Pipeline"]

  PIPE --> EMB["Embedding / Indexing"]
  EMB --> VDB["Vector Database"]

  PIPE --> METRIC["Tính chỉ số học tập"]
  METRIC --> RISK["Risk Score realtime"]
  RISK --> REGROUP["Dynamic Re-Grouping"]
  REGROUP --> CORE["Cập nhật LMS Core<br/>Nhóm sinh viên, trạng thái CSSV,<br/>thông báo, dashboard"]

  VDB --> RAG["RAG Service"]
  RAG --> LLM["LLM API / Local LLM"]

  LLM --> ASSIST["AI Assistant"]
  ASSIST --> ASSIST_OUT["Tra cứu hồ sơ<br/>Tóm tắt lịch sử<br/>Sinh kịch bản tư vấn<br/>Gợi ý phụ đạo"]

  LLM --> REPORT["AI báo cáo & dự đoán"]
  REPORT --> REPORT_OUT["Báo cáo định kỳ<br/>Dự đoán fail môn<br/>Dự đoán bảo lưu<br/>Dự đoán PLO/CLO"]

  LLM --> GRADING["AI chấm BTVN / Project"]
  GRADING --> GRADING_OUT["Nhận xét bài làm<br/>Chấm điểm gợi ý<br/>Phát hiện vấn đề học tập"]

  LLM --> TEACHING["AI đánh giá giảng dạy"]
  TEACHING --> TEACHING_OUT["Phân tích chất lượng giảng dạy<br/>Gợi ý cải thiện"]
```
