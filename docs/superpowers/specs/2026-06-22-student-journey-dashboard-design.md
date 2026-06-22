# Tài liệu thiết kế: Bảng điều khiển Chế độ kép (Dual-Mode Toggle Dashboard) LMS AI

Tài liệu này đặc tả cấu trúc thiết kế, giao diện tương tác và các thay đổi kỹ thuật để cải tiến tệp `index.html` của dự án LMS AI từ một báo cáo quản lý dự án đơn thuần thành một bảng điều khiển tích hợp chế độ kép: Góc Kỹ Thuật (cho Ban Quản Trị) và Trình Giả Lập LMS (cho Sinh Viên).

---

## 1. Bố cục Giao diện Đề xuất (Layout Design)

Giao diện sẽ được tích hợp bộ chuyển đổi (Mode Switcher) nổi bật ở đầu trang (Header hoặc phần đầu Main). Khi chuyển đổi chế độ, các lớp CSS thích hợp sẽ được áp dụng lên `body` hoặc vùng nội dung chính để thay đổi cách hiển thị của Sidebar (aside) và nội dung chính (main).

### 1.1 Chế độ Góc Kỹ Thuật (Technical View Mode)
*   **Mục tiêu**: Giữ lại toàn bộ thông tin kiến trúc kỹ thuật hiện tại nhằm phục vụ cho ban quản trị và nhà phát triển.
*   **Sidebar**: Hiển thị các liên kết điều hướng kỹ thuật:
    1.  Tổng quan kế hoạch
    2.  Kiến trúc hệ thống (Sơ đồ SVG tương tác)
    3.  Lộ trình triển khai (Sprints & API Endpoints)
    4.  Cây chức năng hệ thống
    5.  Quy trình Cố vấn CTSV & Phân loại
    6.  Khảo sát & Giải pháp cho hệ thống cũ (6 Vấn đề & Giải pháp đã viết ở bản cũ)
*   **Nội dung chính**: Hiển thị các section kỹ thuật gốc (kiến trúc, roadmap, function tree, quy trình CTSV, Jira Board ban đầu).

### 1.2 Chế độ Trình Giả Lập LMS (Student Journey Simulator Mode)
*   **Mục tiêu**: Mô phỏng trực quan 6 nhóm tính năng từ tài liệu `LMS_AI_Student.xlsx`.
*   **Sidebar**: Chuyển đổi thành Menu chức năng của LMS Sinh viên:
    1.  🔔 1. Thông tin & Bảng xếp hạng
    2.  🏠 2. Trang chủ & Lộ trình học (Roadmap)
    3.  📖 3. Học tập cá nhân hóa & AI Rika
    4.  ⚡ 4. Động lực học (Gamification)
    5.  🏛️ 5. Hành chính (Một cửa)
    6.  💼 6. Portfolio cá nhân
*   **Nội dung chính**: Hiển thị giao diện giả lập (Mock UI) của từng tính năng với độ tương tác cao:
    *   **Tab 1 - Thông tin**: Giao diện Bảng xếp hạng (Leaderboard) lọc theo hệ đào tạo/khóa/lớp và hòm tin tức đào tạo.
    *   **Tab 2 - Trang chủ & Lộ trình**: Danh sách công việc cần làm hôm nay (Today's Tasks), Lộ trình học khớp với nhu cầu thị trường, Bảng theo dõi Streak học tập, Level & XP.
    *   **Tab 3 - Học tập & AI Rika**: Bài học video/đọc kèm câu hỏi kiểm tra, và khung chat tương tác với Trợ lý AI Rika (hỗ trợ câu hỏi nhanh như tóm tắt bài giảng, giải thích code, sinh câu hỏi luyện tập).
    *   **Tab 4 - Động lực**: Bảng nhiệm vụ (Missions), danh hiệu (Badges) đã đạt được và các giải đấu định kỳ.
    *   **Tab 5 - Hành chính**: Form nộp đơn xin phép/phúc khảo trực tuyến. Danh sách đơn đã nộp hiển thị trạng thái xử lý tự động (Pending &rarr; Approved bởi Rika).
    *   **Tab 6 - Portfolio**: Portfolio tự động tích hợp link dự án Github, danh sách kỹ năng đã đạt, badge và nhận xét giảng viên.

---

## 2. Giải pháp Kỹ thuật & Tương tác (Technical Solutions)

### 2.1 Quản lý Trạng thái (State Management) trong JavaScript
Sử dụng các biến trạng thái toàn cục đơn giản trong tệp `index.html`:
```javascript
let appMode = 'tech'; // 'tech' hoặc 'student'
let activeStudentTab = 'info'; // 'info', 'dashboard', 'learning', 'motivation', 'admin', 'portfolio'
```

### 2.2 Bộ Chuyển đổi Chế độ (Mode Toggle Switch)
Tạo nút toggle bằng CSS Glassmorphism kết hợp hiệu ứng trượt mượt mà ở đầu trang:
```html
<div class="mode-toggle-container">
  <button id="btn-mode-tech" class="mode-btn active" onclick="switchMode('tech')">⚙️ Góc Kỹ Thuật</button>
  <button id="btn-mode-student" class="mode-btn" onclick="switchMode('student')">🎓 Trình Giả Lập LMS</button>
</div>
```

### 2.3 Mô phỏng AI Chatbot Rika (AI Assistant Simulator)
Khung chat với AI Rika sẽ hỗ trợ sinh viên hỏi các câu hỏi mẫu có sẵn. Khi sinh viên click vào một câu hỏi (ví dụ: "Tóm tắt bài học SQL"), chatbot sẽ mô phỏng hiệu ứng hiển thị chữ chạy (typing effect) cực kỳ sinh động để đưa ra phản hồi thông minh tương ứng từ bộ cơ sở dữ liệu mẫu.

### 2.4 Cổng dịch vụ một cửa (One-gate) hoạt động động
Sinh viên có thể chọn loại đơn (xin nghỉ phép, gia hạn bài tập, phúc khảo bài thi...), nhập lý do và bấm gửi. Đơn sẽ ngay lập tức được thêm vào bảng "Lịch sử đơn từ" bên dưới dưới dạng trạng thái **Đang xử lý**, sau đó sau 3 giây sẽ tự động chuyển sang **Đã duyệt** bởi **AI Rika** kèm thông báo in-app.

---

## 3. Kế hoạch Thay đổi Chi tiết (Proposed Changes)

### [MODIFY] [index.html](file:///c:/Users/ADMIN/Desktop/LMS_AI/index.html)
*   **CSS Styles (Dòng 11 - 1206)**:
    *   Thêm các biến CSS mới cho màu sắc chế độ sinh viên (tông màu tím/neon trẻ trung hơn).
    *   Cấu hình các class ẩn/hiển thị dựa trên trạng thái `.mode-tech` và `.mode-student` của thẻ `body`.
    *   Thiết kế giao diện cho các widget giả lập: Leaderboard, Streak, Roadmap timeline, chatbox, missions list, request table, portfolio card.
*   **HTML Structure (Dòng 1208 - 2519)**:
    *   Bổ sung bộ chuyển đổi chế độ ở đầu trang.
    *   Bọc các section kỹ thuật cũ trong một container chung (`div#tech-content-wrapper`).
    *   Thêm container mới (`div#student-simulator-wrapper`) chứa giao diện giả lập LMS Sinh viên với 6 phân vùng tính năng tương ứng.
*   **JavaScript Logic (Dòng 2520 trở đi)**:
    *   Viết hàm `switchMode(mode)` để bật/tắt hiển thị giữa hai chế độ.
    *   Viết hàm điều hướng các tab của giao diện sinh viên.
    *   Xây dựng logic tương tác cho Bảng xếp hạng (lọc dữ liệu mẫu).
    *   Xây dựng logic mô phỏng chat Rika (hiệu ứng typing).
    *   Xây dựng logic thêm đơn từ mới và tự động cập nhật trạng thái đơn từ.

---

## 4. Kế hoạch Kiểm thử & Xác minh (Verification Plan)

### 4.1 Kiểm thử thủ công cục bộ
*   Kiểm tra tính tương thích Responsive trên các kích thước màn hình khác nhau (Desktop, Tablet, Mobile).
*   Kiểm tra tính năng chuyển đổi chế độ: Click đổi qua lại giữa Chế độ kỹ thuật và Trình giả lập xem giao diện có chuyển hoàn toàn không.
*   Kiểm tra tính năng tương tác của Trình giả lập:
    *   Bộ lọc Leaderboard hoạt động mượt mà.
    *   Click gửi câu hỏi mẫu cho Rika hoạt động và hiển thị chữ chạy đúng kịch bản.
    *   Form Một cửa cho phép submit đơn, đơn xuất hiện trong bảng và tự động cập nhật trạng thái sau 3 giây.
    *   Kiểm tra các tab Gamification, Roadmap và Portfolio hiển thị dữ liệu đẹp mắt không có lỗi JavaScript console.

### 4.2 Kiểm thử Triển khai (Vercel deployment)
*   Sử dụng lệnh `npx vercel` để deploy bản cập nhật lên production.
*   Truy cập link Vercel và thực hiện lại các bài test tương tác trên môi trường thực tế.

---
Tài liệu được soạn thảo bởi AI Assistant Antigravity. Vui lòng phê duyệt để chuyển sang bước lập kế hoạch thực thi chi tiết.
