# LMS AI Student Journey Dual-Mode Dashboard Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Remake `index.html` into a dual-mode dashboard containing a Technical Project View and an interactive Student Journey Simulator based on `LMS_AI_Student.xlsx`.

**Architecture:** Implement a global toggle header to dynamically switch between `.mode-tech` and `.mode-student` states on the body element. Toggle matching sidebar navigation and content wrappers. Use vanilla JS to manage states, filters, request forms, and typing chatbot simulation.

**Tech Stack:** HTML5, CSS3 (Vanilla), JavaScript (ES6), Vercel CLI.

---

### Task 1: CSS Styling for Mode Toggle and Simulator Elements

**Files:**
- Modify: `c:\Users\ADMIN\Desktop\LMS_AI\index.html` (inside the `<style>` block, around line 1200, before the `</style>` tag)

- [ ] **Step 1: Write CSS variables and helper classes for dual-mode switching**
  Define styles that hide and show containers based on the `body` class (`.mode-tech` or `.mode-student`).
  Add these CSS rules:
  ```css
  /* Dual-mode visibility controllers */
  body.mode-tech #tech-content-wrapper,
  body.mode-tech aside #tech-sidebar-menu {
    display: flex !important;
  }
  body.mode-tech #student-simulator-wrapper,
  body.mode-tech aside #student-sidebar-menu {
    display: none !important;
  }
  body.mode-student #tech-content-wrapper,
  body.mode-student aside #tech-sidebar-menu {
    display: none !important;
  }
  body.mode-student #student-simulator-wrapper,
  body.mode-student aside #student-sidebar-menu {
    display: flex !important;
  }

  /* Global Mode Toggle Switch Styles */
  .global-header {
    width: calc(100% - var(--toc-width));
    margin-left: var(--toc-width);
    position: fixed;
    top: 0;
    left: 0;
    height: 70px;
    background: rgba(9, 13, 22, 0.4);
    backdrop-filter: blur(20px);
    border-bottom: 1px solid var(--border-color);
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 4rem;
    z-index: 99;
  }
  @media (max-width: 1024px) {
    .global-header {
      width: 100%;
      margin-left: 0;
      padding: 0 2rem;
    }
    main {
      margin-top: 70px !important;
    }
  }
  main {
    margin-top: 70px;
  }
  .mode-switch-wrapper {
    display: flex;
    background: rgba(0, 0, 0, 0.3);
    border: 1px solid var(--border-color);
    border-radius: 30px;
    padding: 4px;
  }
  .mode-switch-btn {
    border: none;
    background: transparent;
    color: var(--text-muted);
    font-family: 'Outfit', sans-serif;
    font-weight: 600;
    padding: 0.5rem 1.25rem;
    border-radius: 20px;
    cursor: pointer;
    transition: all 0.25s ease;
    font-size: 0.85rem;
  }
  .mode-switch-btn.active {
    background: var(--primary);
    color: var(--bg-dark);
    box-shadow: 0 0 12px var(--primary-glow);
  }
  .mode-switch-btn.active.student-color {
    background: var(--secondary);
    color: var(--bg-dark);
    box-shadow: 0 0 12px var(--secondary-glow);
  }

  /* Simulator-specific styles */
  .sim-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 2.5rem;
  }
  .sim-tab-content {
    display: none;
  }
  .sim-tab-content.active {
    display: block;
  }
  /* Leaderboard Styling */
  .leaderboard-table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 1rem;
    text-align: left;
  }
  .leaderboard-table th, .leaderboard-table td {
    padding: 0.85rem 1rem;
    border-bottom: 1px solid var(--border-color);
  }
  .leaderboard-table th {
    color: var(--text-muted);
    font-weight: 600;
  }
  .leaderboard-table tr:hover td {
    background: rgba(255, 255, 255, 0.02);
  }
  .rank-badge {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    display: inline-flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
    font-size: 0.8rem;
  }
  .rank-1 { background: #f59e0b; color: #fff; }
  .rank-2 { background: #cbd5e1; color: #0f172a; }
  .rank-3 { background: #b45309; color: #fff; }

  /* Chatbox Styling */
  .chat-container {
    display: flex;
    flex-direction: column;
    height: 450px;
    background: rgba(0,0,0,0.3);
    border: 1px solid var(--border-color);
    border-radius: 12px;
    overflow: hidden;
  }
  .chat-messages {
    flex-grow: 1;
    padding: 1.25rem;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }
  .chat-msg {
    max-width: 80%;
    padding: 0.85rem 1.1rem;
    border-radius: 12px;
    font-size: 0.9rem;
    line-height: 1.45;
  }
  .chat-msg.student {
    align-self: flex-end;
    background: var(--primary-glow);
    border: 1px solid rgba(56, 189, 248, 0.3);
    color: var(--text-main);
  }
  .chat-msg.rika {
    align-self: flex-start;
    background: var(--secondary-glow);
    border: 1px solid rgba(192, 132, 252, 0.3);
    color: var(--text-main);
  }
  .chat-suggestions {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
    padding: 1rem;
    background: rgba(0,0,0,0.15);
    border-top: 1px solid var(--border-color);
  }
  .chat-suggest-btn {
    background: rgba(255,255,255,0.05);
    border: 1px solid var(--border-color);
    color: var(--text-muted);
    padding: 0.4rem 0.8rem;
    border-radius: 20px;
    font-size: 0.8rem;
    cursor: pointer;
    transition: all 0.2s ease;
  }
  .chat-suggest-btn:hover {
    background: var(--secondary-glow);
    border-color: var(--secondary);
    color: var(--text-main);
  }
  ```

- [ ] **Step 2: Commit CSS changes**
  Run: `git add index.html`
  Run: `git commit -m "feat: add CSS styles for dual-mode dashboard and student journey widgets"`

---

### Task 2: HTML Layout Structures for Mode Toggle and Sidebar Navigation

**Files:**
- Modify: `c:\Users\ADMIN\Desktop\LMS_AI\index.html` (Around line 1208 to 1320)

- [ ] **Step 1: Replace existing Sidebar aside structure to host two separate menus**
  Update the `aside` tag to separate the menus.
  ```html
  <aside>
    <div class="toc-brand">
      <span class="toc-title">LMS AI</span>
      <span class="toc-subtitle" id="aside-subtitle">Kế Hoạch Triển Khai</span>
    </div>
    
    <!-- Menu Chế độ Kỹ thuật (Technical Sidebar) -->
    <ul class="toc-menu" id="tech-sidebar-menu" style="display: flex;">
      <li><a href="#tong-quan" class="toc-link active" onclick="setActiveLink(this)"><span>📝</span><span>1. Tổng quan kế hoạch</span></a></li>
      <li><a href="#kien-truc" class="toc-link" onclick="setActiveLink(this)"><span>📐</span><span>2. Kiến trúc hệ thống</span></a></li>
      <li><a href="#loi-trinh" class="toc-link" onclick="setActiveLink(this)"><span>📅</span><span>3. Lộ trình triển khai</span></a></li>
      <li><a href="#chuc-nang" class="toc-link" onclick="setActiveLink(this)"><span>🌿</span><span>4. Cây chức năng</span></a></li>
      <li><a href="#cssv" class="toc-link" onclick="setActiveLink(this)"><span>👥</span><span>5. Quy trình CSSV</span></a></li>
      <li><a href="#ai-pipeline" class="toc-link" onclick="setActiveLink(this)"><span>⚡</span><span>6. AI Services Pipeline</span></a></li>
      <li><a href="#quan-ly-du-an" class="toc-link" onclick="setActiveLink(this)"><span>📊</span><span>7. Bảng Quản Lý Dự Án</span></a></li>
    </ul>

    <!-- Menu Chế độ Sinh viên (Student Sidebar) -->
    <ul class="toc-menu" id="student-sidebar-menu" style="display: none;">
      <li><a href="javascript:void(0)" class="toc-link active" onclick="switchStudentTab('info', this)"><span>🔔</span><span>1. Thông tin & Bảng xếp hạng</span></a></li>
      <li><a href="javascript:void(0)" class="toc-link" onclick="switchStudentTab('dashboard', this)"><span>🏠</span><span>2. Trang chủ & Lộ trình</span></a></li>
      <li><a href="javascript:void(0)" class="toc-link" onclick="switchStudentTab('learning', this)"><span>📖</span><span>3. Học tập & AI Rika</span></a></li>
      <li><a href="javascript:void(0)" class="toc-link" onclick="switchStudentTab('motivation', this)"><span>⚡</span><span>4. Động lực học tập</span></a></li>
      <li><a href="javascript:void(0)" class="toc-link" onclick="switchStudentTab('admin', this)"><span>🏛️</span><span>5. Hành chính Một cửa</span></a></li>
      <li><a href="javascript:void(0)" class="toc-link" onclick="switchStudentTab('portfolio', this)"><span>💼</span><span>6. Portfolio cá nhân</span></a></li>
    </ul>

    <div class="toc-footer">
      <span id="aside-footer-ver">LMS AI Dashboard - Production v1.2</span>
    </div>
  </aside>
  ```

- [ ] **Step 2: Add Global Header with Mode Toggle**
  Add this block directly after `</aside>` and before `<main>`:
  ```html
  <div class="global-header">
    <div style="font-weight: 600; font-size: 1.1rem; color: #fff;">
      Hệ thống LMS AI thế hệ mới
    </div>
    <div class="mode-switch-wrapper">
      <button id="btn-mode-tech" class="mode-switch-btn active" onclick="switchMode('tech')">⚙️ Góc Kỹ Thuật</button>
      <button id="btn-mode-student" class="mode-switch-btn" onclick="switchMode('student')">🎓 Trình Giả Lập LMS</button>
    </div>
  </div>
  ```

- [ ] **Step 3: Wrap technical sections in `#tech-content-wrapper`**
  Find the beginning of the `<main>` tag. Wrap all original sections (from `#tong-quan` to `#quan-ly-du-an`) inside:
  `<div id="tech-content-wrapper" style="display: flex; flex-direction: column; gap: 5rem; width: 100%;">`
  And close this div right before the end of the `</main>` tag.

- [ ] **Step 4: Commit structural changes**
  Run: `git add index.html`
  Run: `git commit -m "feat: restructure HTML layout with global mode switcher and wrappers"`

---

### Task 3: HTML Structure for Student Journey Simulator Tabs

**Files:**
- Modify: `c:\Users\ADMIN\Desktop\LMS_AI\index.html` (inside `<main>`, after `#tech-content-wrapper` and before `</main>`)

- [ ] **Step 1: Write structural container for student simulator tabs**
  Create `#student-simulator-wrapper` with all the tabbed layouts.
  ```html
  <div id="student-simulator-wrapper" style="display: none; flex-direction: column; gap: 3rem; width: 100%;">
    
    <!-- Tab 1: Thông tin & Bảng xếp hạng -->
    <div id="tab-info" class="sim-tab-content active">
      <div class="section-header">
        <span class="section-badge">Thông tin</span>
        <h2 class="section-title">Bảng Xếp Hạng Năng Lực Học Tập</h2>
        <p class="section-desc">Theo dõi thứ hạng học tập toàn diện dựa trên điểm số chuyên môn và ý thức học tập (R-point/XP).</p>
      </div>
      <div class="card" style="margin-bottom: 2rem;">
        <div style="display: flex; gap: 1rem; margin-bottom: 1.5rem; flex-wrap: wrap;">
          <div>
            <label class="label" style="display:block; margin-bottom:0.35rem;">Hệ Đào Tạo</label>
            <select id="leaderboard-program" onchange="filterLeaderboard()" class="mock-input" style="background:#111827; color:#fff; border:1px solid var(--border-color); padding:0.5rem; border-radius:6px;">
              <option value="all">Tất cả hệ học</option>
              <option value="clc">Chất lượng cao</option>
              <option value="chuan">Hệ chuẩn (Đại trà)</option>
            </select>
          </div>
          <div>
            <label class="label" style="display:block; margin-bottom:0.35rem;">Khóa học</label>
            <select id="leaderboard-cohort" onchange="filterLeaderboard()" class="mock-input" style="background:#111827; color:#fff; border:1px solid var(--border-color); padding:0.5rem; border-radius:6px;">
              <option value="all">Tất cả các khóa</option>
              <option value="k24">Khóa K24</option>
              <option value="k25">Khóa K25</option>
            </select>
          </div>
        </div>
        
        <table class="leaderboard-table">
          <thead>
            <tr>
              <th>Hạng</th>
              <th>Mã Sinh Viên</th>
              <th>Họ và Tên</th>
              <th>Lớp</th>
              <th>Hệ học</th>
              <th>Tích lũy XP</th>
              <th>Cấp độ (Level)</th>
            </tr>
          </thead>
          <tbody id="leaderboard-tbody">
            <!-- Dynamically populated via JS -->
          </tbody>
        </table>
      </div>

      <div class="section-header" style="margin-top: 3rem;">
        <h3 class="section-title" style="font-size: 1.5rem;">🔔 Thông Báo Đào Tạo Từ Hệ Thống</h3>
      </div>
      <div style="display:grid; grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); gap:1.5rem;">
        <div class="card" style="border-left: 4px solid var(--primary); padding: 1.5rem;">
          <span class="label" style="color:var(--primary);">Hệ thống đào tạo</span>
          <h4 style="margin: 0.5rem 0; font-size:1.1rem;">Lịch đăng ký môn học học kỳ phụ sắp mở</h4>
          <p style="font-size:0.85rem; color:var(--text-muted); line-height:1.4;">Bắt đầu đăng ký học phần tương đương từ ngày 25/06/2026. Sinh viên chú ý cập nhật IndividualStudyPlan.</p>
        </div>
        <div class="card" style="border-left: 4px solid var(--secondary); padding: 1.5rem;">
          <span class="label" style="color:var(--secondary);">Khóa học Web nâng cao</span>
          <h4 style="margin: 0.5rem 0; font-size:1.1rem;">AI Project Agent đã cập nhật Sprints đầu bài</h4>
          <p style="font-size:0.85rem; color:var(--text-muted); line-height:1.4;">Hãy truy cập mục Quản lý Dự án để nhận danh sách công việc chi tiết được AI phân rã cho Đồ án nhóm của bạn.</p>
        </div>
      </div>
    </div>

    <!-- Tab 2: Trang chủ & Lộ trình -->
    <div id="tab-dashboard" class="sim-tab-content">
      <div class="section-header">
        <span class="section-badge">Trang chủ</span>
        <h2 class="section-title">Dashboard Sinh Viên</h2>
        <p class="section-desc">Giao diện tổng hợp công việc cần làm hôm nay, tiến độ lộ trình học tập và trạng thái Streak.</p>
      </div>

      <div class="overview-grid" style="grid-template-columns: 1.8fr 1.2fr; gap: 2rem;">
        <!-- Left: Today Tasks & Roadmap -->
        <div style="display:flex; flex-direction:column; gap:2rem;">
          <div class="card">
            <h3 style="font-size: 1.2rem; margin-bottom: 1rem; color: var(--primary);">✅ Công việc cần làm hôm nay</h3>
            <div style="display:flex; flex-direction:column; gap:0.75rem;">
              <label style="display:flex; align-items:center; gap:0.75rem; font-size:0.95rem; cursor:pointer;">
                <input type="checkbox" checked disabled> <span style="text-decoration:line-through; color:var(--text-muted);">Xem Video bài giảng: Buổi 5 - Thiết kế RDB và chuẩn hóa DB</span>
              </label>
              <label style="display:flex; align-items:center; gap:0.75rem; font-size:0.95rem; cursor:pointer;">
                <input type="checkbox" onchange="toggleTaskCheck(this)"> <span>Làm 5 câu trắc nghiệm kiểm chứng lý thuyết SQL Joins</span>
              </label>
              <label style="display:flex; align-items:center; gap:0.75rem; font-size:0.95rem; cursor:pointer;">
                <input type="checkbox" onchange="toggleTaskCheck(this)"> <span>Nộp báo cáo Sprint 1 Đồ án đăng ký môn học lên Kanban</span>
              </label>
            </div>
          </div>

          <div class="card">
            <h3 style="font-size: 1.2rem; margin-bottom: 1rem; color: var(--secondary);">🗺️ Lộ trình đào tạo cá nhân hóa (Roadmap)</h3>
            <p style="font-size:0.85rem; color:var(--text-muted); margin-bottom: 1.25rem;">Lộ trình được tự động thiết kế dựa trên năng lực và nhu cầu tuyển dụng Fullstack Web Developer:</p>
            <div style="display:flex; flex-direction:column; gap:1rem; position:relative; padding-left:1.5rem; border-left: 2px solid var(--border-color);">
              <div style="position:relative;">
                <span style="position:absolute; left:-2rem; top:0.2rem; width:12px; height:12px; border-radius:50%; background:var(--success); box-shadow:0 0 6px var(--success);"></span>
                <h4 style="font-size:0.95rem; color:#fff;">Giai đoạn 1: HTML5, CSS3 & Javascript cơ bản</h4>
                <p style="font-size:0.8rem; color:var(--success);">✔ Đã hoàn thành - 100%</p>
              </div>
              <div style="position:relative;">
                <span style="position:absolute; left:-2rem; top:0.2rem; width:12px; height:12px; border-radius:50%; background:var(--primary); box-shadow:0 0 6px var(--primary);"></span>
                <h4 style="font-size:0.95rem; color:#fff;">Giai đoạn 2: Cơ sở dữ liệu quan hệ SQL & API với ExpressJS</h4>
                <p style="font-size:0.8rem; color:var(--primary);">⏳ Đang học - Đạt 65% tiến trình</p>
              </div>
              <div style="position:relative;">
                <span style="position:absolute; left:-2rem; top:0.2rem; width:12px; height:12px; border-radius:50%; background:rgba(255,255,255,0.2);"></span>
                <h4 style="font-size:0.95rem; color:var(--text-muted);">Giai đoạn 3: Framework React & Next.js nâng cao</h4>
                <p style="font-size:0.8rem; color:var(--text-muted);">🔒 Chưa mở khóa (Dự kiến mở vào tuần tới)</p>
              </div>
            </div>
          </div>
        </div>

        <!-- Right: Student Stats & Streak -->
        <div style="display:flex; flex-direction:column; gap:2rem;">
          <div class="card" style="text-align:center;">
            <div style="width: 80px; height: 80px; border-radius: 50%; background: var(--secondary-glow); border: 2px solid var(--secondary); display:flex; align-items:center; justify-content:center; font-size:2rem; margin: 0 auto 1rem auto;">👨‍💻</div>
            <h3 style="font-size: 1.25rem;">Nguyễn Văn A</h3>
            <p style="font-size: 0.8rem; color:var(--text-muted);">Lớp: D21HTTT01 - Hệ Chất lượng cao</p>
            <div style="display:flex; justify-content:space-around; margin-top:1.5rem; border-top:1px solid var(--border-color); padding-top:1rem;">
              <div>
                <span style="font-size: 1.5rem; font-weight:700; color:var(--primary);">8,450</span>
                <span style="display:block; font-size:0.7rem; color:var(--text-muted); text-transform:uppercase;">Tổng XP</span>
              </div>
              <div>
                <span style="font-size: 1.5rem; font-weight:700; color:var(--secondary);">Level 14</span>
                <span style="display:block; font-size:0.7rem; color:var(--text-muted); text-transform:uppercase;">Cấp độ</span>
              </div>
            </div>
          </div>

          <div class="card" style="text-align:center; border: 1px solid rgba(251, 191, 36, 0.3); background: rgba(251, 191, 36, 0.02);">
            <div style="font-size: 2.5rem; margin-bottom: 0.5rem; animation: pulse 2s infinite;">🔥</div>
            <h3 style="font-size: 1.2rem; color: var(--warning);">Streak Học Tập: <span id="streak-count">5</span> Ngày Liên Tiếp</h3>
            <p style="font-size: 0.8rem; color:var(--text-muted); margin-bottom:1rem; line-height:1.4;">Duy trì học tập hàng ngày để nhân hệ số XP và nhận badge đặc biệt.</p>
            <button class="ai-pm-btn" style="background:var(--warning); color:#000;" id="btn-streak-checkin" onclick="triggerStreakCheckin()">🔥 Điểm danh học tập ngay hôm nay</button>
          </div>
        </div>
      </div>
    </div>

    <!-- Tab 3: Học tập & AI Rika -->
    <div id="tab-learning" class="sim-tab-content">
      <div class="section-header">
        <span class="section-badge">Học tập</span>
        <h2 class="section-title">Học tập Cá nhân hóa & Trợ lý ảo Rika</h2>
        <p class="section-desc">Học tập theo Session/Lesson thiết kế riêng và trao đổi hỏi đáp 24/7 với AI Assistant.</p>
      </div>

      <div class="overview-grid" style="grid-template-columns: 1fr 1.2fr; gap: 2rem;">
        <!-- Left: Course Lessons -->
        <div class="card">
          <h3 style="font-size: 1.2rem; margin-bottom: 1.25rem;">📖 Giáo trình: Thiết kế Web nâng cao</h3>
          <div style="display:flex; flex-direction:column; gap:0.75rem;">
            <div style="padding:0.75rem; background:rgba(255,255,255,0.03); border:1px solid var(--border-color); border-radius:8px;">
              <h4 style="font-size:0.9rem; margin-bottom:0.25rem;">Lesson 5.1: Video giảng dạy SQL Joins</h4>
              <span style="font-size:0.7rem; color:var(--success);">✔ Đã xem (12 phút)</span>
            </div>
            <div style="padding:0.75rem; background:rgba(56,189,248,0.05); border:1px solid var(--primary); border-radius:8px;">
              <h4 style="font-size:0.9rem; margin-bottom:0.25rem; color:var(--primary);">Lesson 5.2: Bài thực hành viết Query phức hợp</h4>
              <span style="font-size:0.7rem; color:var(--primary);">⏳ Đang học - Yêu cầu chạy testcases</span>
            </div>
            <div style="padding:0.75rem; background:rgba(255,255,255,0.01); border:1px solid var(--border-color); border-radius:8px; opacity:0.6;">
              <h4 style="font-size:0.9rem; margin-bottom:0.25rem;">Lesson 5.3: Quiz củng cố kiến thức môn học</h4>
              <span style="font-size:0.7rem; color:var(--text-muted);">🔒 Chưa mở khóa (Xem xong video)</span>
            </div>
          </div>
        </div>

        <!-- Right: AI Chat Rika -->
        <div class="chat-container">
          <div style="padding: 1rem; background: rgba(0,0,0,0.2); border-bottom: 1px solid var(--border-color); display:flex; justify-content:space-between; align-items:center;">
            <div style="display:flex; align-items:center; gap:0.5rem;">
              <div style="width:12px; height:12px; background:var(--success); border-radius:50%;"></div>
              <strong style="color:var(--secondary)">🤖 Trợ lý học tập Rika (AI)</strong>
            </div>
            <span style="font-size:0.75rem; color:var(--text-muted);">Hỏi đáp RAG & VectorDB</span>
          </div>

          <div class="chat-messages" id="rika-chat-messages">
            <div class="chat-msg rika">
              Xin chào! Mình là <strong>Rika</strong>. Mình đã nắm được tài liệu và slide của môn học <strong>Thiết kế Web nâng cao</strong> của lớp bạn. Bạn có điều gì cần mình hỗ trợ giải đáp không?
            </div>
          </div>

          <div class="chat-suggestions">
            <button class="chat-suggest-btn" onclick="sendRikaSample('sql_join')">💡 Giải thích lại SQL Outer Join dễ hiểu hơn</button>
            <button class="chat-suggest-btn" onclick="sendRikaSample('code_debug')">💡 Xem gợi ý sửa lỗi code (Express Server)</button>
            <button class="chat-suggest-btn" onclick="sendRikaSample('summary')">💡 Tóm tắt bài học SQL tuần này</button>
            <button class="chat-suggest-btn" onclick="sendRikaSample('practice')">💡 Sinh câu hỏi trắc nghiệm tự luyện</button>
          </div>
        </div>
      </div>
    </div>

    <!-- Tab 4: Động lực học tập -->
    <div id="tab-motivation" class="sim-tab-content">
      <div class="section-header">
        <span class="section-badge purple">Động lực</span>
        <h2 class="section-title">Hệ Thống Gamification & Các Cuộc Thi</h2>
        <p class="section-desc">Tạo động lực học tập thông qua nhiệm vụ hàng ngày, huy hiệu đặc biệt và các cuộc thi học thuật.</p>
      </div>

      <div class="overview-grid" style="grid-template-columns: 1fr 1fr; gap: 2rem;">
        <!-- Missions list -->
        <div class="card">
          <h3 style="font-size:1.2rem; margin-bottom:1.25rem; color:var(--secondary);">⚡ Nhiệm vụ tuần này</h3>
          <div style="display:flex; flex-direction:column; gap:1rem;">
            <div style="display:flex; justify-content:space-between; align-items:center; padding-bottom:0.75rem; border-bottom:1px solid var(--border-color);">
              <div>
                <h4 style="font-size:0.95rem; margin-bottom:0.15rem;">Chăm chỉ học tập</h4>
                <p style="font-size:0.75rem; color:var(--text-muted);">Xem trọn vẹn 3 video bài giảng mới</p>
              </div>
              <span style="font-size:0.85rem; color:var(--success); font-weight:600;">+250 XP</span>
            </div>
            <div style="display:flex; justify-content:space-between; align-items:center; padding-bottom:0.75rem; border-bottom:1px solid var(--border-color);">
              <div>
                <h4 style="font-size:0.95rem; margin-bottom:0.15rem;">Chúa tể Query</h4>
                <p style="font-size:0.75rem; color:var(--text-muted);">Hoàn thành 1 bài tập SQL Join không lỗi</p>
              </div>
              <span style="font-size:0.85rem; color:var(--success); font-weight:600;">+500 XP</span>
            </div>
            <div style="display:flex; justify-content:space-between; align-items:center;">
              <div>
                <h4 style="font-size:0.95rem; margin-bottom:0.15rem;">Nhà nghiên cứu thông thái</h4>
                <p style="font-size:0.75rem; color:var(--text-muted);">Đặt 2 câu hỏi học vụ cho trợ lý AI Rika</p>
              </div>
              <span style="font-size:0.85rem; color:var(--success); font-weight:600;">+100 XP</span>
            </div>
          </div>
        </div>

        <!-- Badges box -->
        <div class="card">
          <h3 style="font-size:1.2rem; margin-bottom:1.25rem; color:var(--primary);">🏅 Huy hiệu của tôi</h3>
          <div style="display:flex; gap:1rem; flex-wrap:wrap; justify-content:center; padding: 1rem 0;">
            <div style="text-align:center; width:80px;">
              <div style="font-size:2.5rem; background:rgba(52,211,153,0.1); border-radius:50%; width:60px; height:60px; display:flex; align-items:center; justify-content:center; margin:0 auto 0.5rem auto;">🚀</div>
              <span style="font-size:0.75rem; display:block; font-weight:500;">Tân Binh</span>
            </div>
            <div style="text-align:center; width:80px;">
              <div style="font-size:2.5rem; background:rgba(56,189,248,0.1); border-radius:50%; width:60px; height:60px; display:flex; align-items:center; justify-content:center; margin:0 auto 0.5rem auto;">🎯</div>
              <span style="font-size:0.75rem; display:block; font-weight:500;">Chuyên Cần</span>
            </div>
            <div style="text-align:center; width:80px;">
              <div style="font-size:2.5rem; background:rgba(251,191,36,0.1); border-radius:50%; width:60px; height:60px; display:flex; align-items:center; justify-content:center; margin:0 auto 0.5rem auto;">🏆</div>
              <span style="font-size:0.75rem; display:block; font-weight:500;">Thủ Khoa</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Tab 5: Hành chính Một cửa -->
    <div id="tab-admin" class="sim-tab-content">
      <div class="section-header">
        <span class="section-badge">Hành chính</span>
        <h2 class="section-title">Trung Tâm Hành Chính 1 Cửa Rika</h2>
        <p class="section-desc">Gửi các yêu cầu hành chính trực tuyến và theo dõi tiến độ phê duyệt tự động của trợ lý Rika.</p>
      </div>

      <div class="overview-grid" style="grid-template-columns: 1fr 1.5fr; gap: 2rem;">
        <!-- Left: Request form -->
        <div class="card">
          <h3 style="font-size:1.2rem; margin-bottom:1.25rem;">✍️ Tạo đơn yêu cầu mới</h3>
          <form id="one-gate-form" onsubmit="submitOneGateRequest(event)">
            <div style="margin-bottom:1rem;">
              <label class="label" style="display:block; margin-bottom:0.35rem;">Loại Đơn Yêu Cầu</label>
              <select id="req-type" class="mock-input" style="width:100%; background:#111827; color:#fff; border:1px solid var(--border-color); padding:0.6rem; border-radius:8px;">
                <option value="Đơn xin nghỉ học tạm thời">Đơn xin nghỉ học tạm thời (Có phép)</option>
                <option value="Đơn xin gia hạn nộp bài tập">Đơn xin gia hạn nộp bài tập đồ án</option>
                <option value="Đơn xin phúc khảo điểm thi">Đơn xin phúc khảo điểm kiểm tra</option>
                <option value="Đơn xin hoãn nộp học phí">Đơn xin hoãn nộp học phí học kỳ</option>
              </select>
            </div>
            <div style="margin-bottom:1.25rem;">
              <label class="label" style="display:block; margin-bottom:0.35rem;">Lý do chi tiết</label>
              <textarea id="req-reason" required placeholder="Nhập lý do chi tiết..." class="mock-input" style="width:100%; height:100px; background:#111827; color:#fff; border:1px solid var(--border-color); padding:0.6rem; border-radius:8px; font-family:'Outfit', sans-serif; resize:none;"></textarea>
            </div>
            <button type="submit" class="ai-pm-btn">🚀 Gửi yêu cầu lên Rika</button>
          </form>
        </div>

        <!-- Right: Request History -->
        <div class="card">
          <h3 style="font-size:1.2rem; margin-bottom:1rem;">📜 Lịch sử xử lý đơn từ</h3>
          <div style="overflow-x:auto;">
            <table class="leaderboard-table" style="font-size:0.85rem;">
              <thead>
                <tr>
                  <th>Mã đơn</th>
                  <th>Loại đơn</th>
                  <th>Người phụ trách</th>
                  <th>Trạng thái</th>
                  <th>Thời gian</th>
                </tr>
              </thead>
              <tbody id="one-gate-history-tbody">
                <tr>
                  <td>REQ-0021</td>
                  <td>Đơn xin nghỉ học có phép</td>
                  <td>🤖 Rika (Cố vấn)</td>
                  <td><span style="background:rgba(52,211,153,0.15); color:var(--success); padding:0.15rem 0.4rem; border-radius:4px; font-weight:600;">Đã duyệt</span></td>
                  <td>18/06/2026</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>

    <!-- Tab 6: Portfolio cá nhân -->
    <div id="tab-portfolio" class="sim-tab-content">
      <div class="section-header">
        <span class="section-badge success">Portfolio</span>
        <h2 class="section-title">Hồ Sơ Năng Lực Học Viên (Portfolio Tự Động)</h2>
        <p class="section-desc">Portfolio tự động đồng bộ hóa thông tin dự án môn học, kỹ năng cốt lõi và đánh giá của giảng viên.</p>
      </div>

      <div class="card" style="display:flex; flex-direction:column; gap:2rem;">
        <div style="display:flex; align-items:center; gap:1.5rem; border-bottom:1px solid var(--border-color); padding-bottom:1.5rem; flex-wrap:wrap;">
          <div style="width:70px; height:70px; border-radius:50%; background:var(--primary-glow); border:2px solid var(--primary); display:flex; align-items:center; justify-content:center; font-size:1.8rem;">👨‍🎓</div>
          <div>
            <h3 style="font-size:1.4rem;">Nguyễn Văn A</h3>
            <p style="font-size:0.85rem; color:var(--text-muted); margin-top:0.15rem;">Lớp D21HTTT01 • Chuyên ngành Hệ thống thông tin quản lý</p>
          </div>
        </div>

        <div style="display:grid; grid-template-columns:1fr 1fr; gap:2rem;">
          <!-- Left: Projects -->
          <div>
            <h4 style="font-size:1.1rem; color:var(--primary); margin-bottom:1rem;">📂 Các dự án đồ án thực hành</h4>
            <div style="display:flex; flex-direction:column; gap:1rem;">
              <div style="background:rgba(255,255,255,0.02); border:1px solid var(--border-color); padding:1rem; border-radius:8px;">
                <h5 style="font-size:0.95rem; margin-bottom:0.25rem;">Hệ thống đăng ký môn học trực tuyến</h5>
                <p style="font-size:0.8rem; color:var(--text-muted); margin-bottom:0.5rem;">ExpressJS, PostgreSQL, ReactJS. Tích hợp AI Project Agent phân rã task nhóm.</p>
                <a href="https://github.com/nguyen-van-a/lms-enrollment-demo" target="_blank" style="color:var(--primary); font-size:0.8rem; text-decoration:none; font-weight:600;">🔗 Github Project Link</a>
              </div>
            </div>
          </div>

          <!-- Right: Skills & Comments -->
          <div style="display:flex; flex-direction:column; gap:1.5rem;">
            <div>
              <h4 style="font-size:1.1rem; color:var(--secondary); margin-bottom:0.75rem;">⚡ Kỹ năng đã tích lũy</h4>
              <div style="display:flex; flex-wrap:wrap; gap:0.5rem;">
                <span style="background:var(--primary-glow); color:var(--primary); border:1px solid rgba(56,189,248,0.2); padding:0.2rem 0.5rem; border-radius:4px; font-size:0.8rem;">SQL Query</span>
                <span style="background:var(--primary-glow); color:var(--primary); border:1px solid rgba(56,189,248,0.2); padding:0.2rem 0.5rem; border-radius:4px; font-size:0.8rem;">NodeJS / Express</span>
                <span style="background:var(--primary-glow); color:var(--primary); border:1px solid rgba(56,189,248,0.2); padding:0.2rem 0.5rem; border-radius:4px; font-size:0.8rem;">UI/UX Design</span>
              </div>
            </div>

            <div style="border-top:1px solid var(--border-color); padding-top:1rem;">
              <h4 style="font-size:1.1rem; color:var(--success); margin-bottom:0.5rem;">💬 Đánh giá từ giảng viên</h4>
              <blockquote style="font-style:italic; font-size:0.85rem; color:var(--text-muted); padding-left:0.75rem; border-left:2px solid var(--success); line-height:1.5;">
                "Sinh viên A có khả năng tự học tốt, tiếp thu nhanh các buổi lý thuyết SQL và tương tác tích cực với trợ lý học vụ AI Rika để tự phát triển kỹ năng coding." - TS. Nguyễn Văn B (Giảng viên Web nâng cao)
              </blockquote>
            </div>
          </div>
        </div>
      </div>
    </div>

  </div>
  ```

- [ ] **Step 2: Commit tab structures**
  Run: `git add index.html`
  Run: `git commit -m "feat: add HTML structures for student journey simulator tabs"`

---

### Task 4: JavaScript Logic for Mode Toggle and Student Tab Navigation

**Files:**
- Modify: `c:\Users\ADMIN\Desktop\LMS_AI\index.html` (inside the `<script>` block, around line 2500)

- [ ] **Step 1: Write switchMode and switchStudentTab functions**
  Add the JavaScript logic to toggle body classes, manage active tabs, and adjust the aside branding metadata.
  ```javascript
  // Application Mode State ('tech' or 'student')
  let appMode = 'tech';
  
  // Switch Global Mode
  function switchMode(mode) {
    appMode = mode;
    document.body.className = mode === 'tech' ? 'mode-tech' : 'mode-student';
    
    // Toggle active switch buttons
    document.getElementById('btn-mode-tech').classList.toggle('active', mode === 'tech');
    document.getElementById('btn-mode-student').classList.toggle('active', mode === 'student');
    
    // Style second color for student mode button
    const studentBtn = document.getElementById('btn-mode-student');
    if (mode === 'student') {
      studentBtn.classList.add('student-color');
      document.getElementById('aside-subtitle').innerText = 'Hành trình sinh viên';
      document.getElementById('aside-footer-ver').innerText = 'LMS AI Simulator v2.0';
      // Load initial mock leaderboard data
      populateLeaderboard();
    } else {
      studentBtn.classList.remove('student-color');
      document.getElementById('aside-subtitle').innerText = 'Kế Hoạch Triển Khai';
      document.getElementById('aside-footer-ver').innerText = 'LMS AI Dashboard - Production v1.2';
    }
  }

  // Switch Student Tabs
  function switchStudentTab(tabName, linkEl) {
    // Hide all tabs
    document.querySelectorAll('.sim-tab-content').forEach(tab => {
      tab.classList.remove('active');
    });
    // Show active tab
    const activeTab = document.getElementById(`tab-${tabName}`);
    if (activeTab) {
      activeTab.classList.add('active');
    }

    // Toggle active link class on sidebar
    document.querySelectorAll('#student-sidebar-menu .toc-link').forEach(link => {
      link.classList.remove('active');
    });
    if (linkEl) {
      linkEl.classList.add('active');
    }
  }

  // Initialize app state to tech mode
  document.addEventListener('DOMContentLoaded', () => {
    switchMode('tech');
  });
  ```

- [ ] **Step 2: Commit toggle logic**
  Run: `git add index.html`
  Run: `git commit -m "feat: add JavaScript toggle logic for dual-mode switching"`

---

### Task 5: Leaderboard Data Filtering Logic (Tab 1)

**Files:**
- Modify: `c:\Users\ADMIN\Desktop\LMS_AI\index.html` (inside the `<script>` block, below switchMode)

- [ ] **Step 1: Write mock data and populateLeaderboard function**
  Define a list of mock students and build the rendering function that supports program and cohort filtering.
  ```javascript
  const mockStudents = [
    { rank: 1, id: 'B21DCCN001', name: 'Trần Minh Quân', class: 'D21CN01', program: 'clc', cohort: 'k24', xp: 9850, lvl: 15 },
    { rank: 2, id: 'B21DCCN102', name: 'Nguyễn Văn A', class: 'D21HTTT01', program: 'clc', cohort: 'k24', xp: 8450, lvl: 14 },
    { rank: 3, id: 'B22DCCN003', name: 'Phạm Thảo Vy', class: 'D22CN02', program: 'chuan', cohort: 'k25', xp: 7200, lvl: 12 },
    { rank: 4, id: 'B22DCCN204', name: 'Lê Hoàng Long', class: 'D22HTTT02', program: 'chuan', cohort: 'k25', xp: 6850, lvl: 11 },
    { rank: 5, id: 'B21DCCN005', name: 'Vũ Minh Thu', class: 'D21CN03', program: 'clc', cohort: 'k24', xp: 6400, lvl: 10 }
  ];

  function populateLeaderboard() {
    const tbody = document.getElementById('leaderboard-tbody');
    if (!tbody) return;
    
    tbody.innerHTML = '';
    const selectedProgram = document.getElementById('leaderboard-program').value;
    const selectedCohort = document.getElementById('leaderboard-cohort').value;

    let rankIndex = 1;
    mockStudents.forEach(student => {
      const matchProgram = selectedProgram === 'all' || student.program === selectedProgram;
      const matchCohort = selectedCohort === 'all' || student.cohort === selectedCohort;

      if (matchProgram && matchCohort) {
        let rankClass = '';
        if (rankIndex === 1) rankClass = 'rank-1';
        else if (rankIndex === 2) rankClass = 'rank-2';
        else if (rankIndex === 3) rankClass = 'rank-3';

        const row = document.createElement('tr');
        row.innerHTML = `
          <td><span class="rank-badge ${rankClass}">${rankIndex}</span></td>
          <td><code>${student.id}</code></td>
          <td>${student.name}</td>
          <td>${student.class}</td>
          <td>${student.program === 'clc' ? 'Chất lượng cao' : 'Hệ chuẩn'}</td>
          <td><strong style="color:var(--success);">${student.xp.toLocaleString()} XP</strong></td>
          <td>Level ${student.lvl}</td>
        `;
        tbody.appendChild(row);
        rankIndex++;
      }
    });
  }

  function filterLeaderboard() {
    populateLeaderboard();
  }
  ```

- [ ] **Step 2: Commit leaderboard filtering changes**
  Run: `git add index.html`
  Run: `git commit -m "feat: add leaderboard data mapping and filtering script"`

---

### Task 6: Streak, Today's Tasks, and AI Chatbot Interactive Simulator

**Files:**
- Modify: `c:\Users\ADMIN\Desktop\LMS_AI\index.html` (inside the `<script>` block)

- [ ] **Step 1: Write Streak check-in and task tracking functions**
  ```javascript
  let streakCheckinDone = false;
  function triggerStreakCheckin() {
    if (streakCheckinDone) {
      alert('Hôm nay bạn đã điểm danh tích lũy rồi!');
      return;
    }
    streakCheckinDone = true;
    const streakCountSpan = document.getElementById('streak-count');
    let currentStreak = parseInt(streakCountSpan.innerText);
    currentStreak += 1;
    streakCountSpan.innerText = currentStreak;

    // Trigger visual style success updates
    const checkinBtn = document.getElementById('btn-streak-checkin');
    checkinBtn.innerText = '🔥 Đã điểm danh hôm nay!';
    checkinBtn.style.background = 'var(--success)';
    checkinBtn.style.color = '#fff';
    checkinBtn.disabled = true;

    alert('Điểm danh thành công! Bạn nhận được +100 XP thưởng.');
  }

  function toggleTaskCheck(checkbox) {
    if (checkbox.checked) {
      alert('Tuyệt vời! Hoàn thành công việc nhận thêm +50 XP.');
    }
  }
  ```

- [ ] **Step 2: Write simulated streaming chatbot response logic**
  Create preloaded chatbot response scripts that type out word-by-word.
  ```javascript
  const rikaScripts = {
    'sql_join': ` Outer Join (Liên kết ngoài) dùng để truy vấn dữ liệu khi có bản ghi ở bảng này nhưng không nhất thiết phải khớp ở bảng kia.
Có 2 loại chính:
1. **Left Outer Join**: Lấy toàn bộ hàng ở bảng bên trái (Left), nếu bảng bên phải không khớp thì điền null.
2. **Right Outer Join**: Lấy toàn bộ hàng ở bảng bên phải (Right), nếu bảng bên trái không khớp thì điền null.

*Ví dụ SQL:*
\`\`\`sql
SELECT SinhVien.HoTen, DiemThi.MonC
FROM SinhVien
LEFT JOIN DiemThi ON SinhVien.MSSV = DiemThi.MSSV;
\`\`\`
*(Nếu sinh viên chưa thi, điểm sẽ tự động trả về NULL)*`,
    
    'code_debug': ` Dựa trên phân tích, file **server.js** của bạn lỗi do thiếu middleware xử lý JSON. Hãy thêm dòng:
\`\`\`javascript
app.use(express.json());
\`\`\`
trước khi định nghĩa các API Endpoints nhé. Điều này giúp tránh lỗi request body trả về undefined.`,
    
    'summary': ` Tóm tắt bài học SQL Tuần này:
- Hiểu cấu trúc lập bảng (Create Table) và các ràng buộc dữ liệu (Primary Key, Foreign Key).
- Hiểu sâu các lệnh JOIN: INNER JOIN (Lấy phần giao), LEFT/RIGHT JOIN (Outer joins).
- Làm quen với các hàm tổng hợp (Aggregate functions): SUM, AVG, COUNT, GROUP BY.`,
    
    'practice': ` Dưới đây là câu hỏi ôn tập nhanh cho bạn:
**Câu hỏi:** Phân biệt kết quả giữa INNER JOIN và LEFT JOIN trong SQL?
A) INNER JOIN lấy tất cả bản ghi của bảng trái, LEFT JOIN chỉ lấy phần giao.
B) INNER JOIN chỉ lấy phần giao dữ liệu, LEFT JOIN lấy tất cả dữ liệu bảng trái bao gồm cả phần không khớp.
C) Cả hai đều cho kết quả giống nhau.

*Nhập đáp án bạn chọn vào khung chat để kiểm tra nhé!*`
  };

  function sendRikaSample(key) {
    const chatMessages = document.getElementById('rika-chat-messages');
    if (!chatMessages) return;

    const requestText = {
      'sql_join': 'Giải thích Outer Join dễ hiểu hơn',
      'code_debug': 'Xem gợi ý sửa lỗi code Express Server',
      'summary': 'Tóm tắt bài học SQL tuần này',
      'practice': 'Sinh câu hỏi trắc nghiệm tự luyện'
    }[key];

    // Append Student Message
    const studentMsg = document.createElement('div');
    studentMsg.className = 'chat-msg student';
    studentMsg.innerText = requestText;
    chatMessages.appendChild(studentMsg);
    chatMessages.scrollTop = chatMessages.scrollHeight;

    // Setup Rika Typing simulation
    setTimeout(() => {
      const rikaMsg = document.createElement('div');
      rikaMsg.className = 'chat-msg rika';
      rikaMsg.innerHTML = '<strong>Rika đang soạn câu trả lời...</strong>';
      chatMessages.appendChild(rikaMsg);
      chatMessages.scrollTop = chatMessages.scrollHeight;

      let responseText = rikaScripts[key];
      let charIndex = 0;
      rikaMsg.innerHTML = '🤖 '; // clear typing placeholder

      const typingTimer = setInterval(() => {
        if (charIndex < responseText.length) {
          rikaMsg.innerHTML += responseText.charAt(charIndex);
          charIndex++;
          chatMessages.scrollTop = chatMessages.scrollHeight;
        } else {
          clearInterval(typingTimer);
        }
      }, 8); // Speed of character typing
    }, 800);
  }
  ```

- [ ] **Step 3: Commit chatbot and streak check-in changes**
  Run: `git add index.html`
  Run: `git commit -m "feat: add interactive streak, task tracking, and AI chatbot simulation"`

---

### Task 7: Interactive One-Gate Request Form with Auto-Approval

**Files:**
- Modify: `c:\Users\ADMIN\Desktop\LMS_AI\index.html` (inside the `<script>` block)

- [ ] **Step 1: Write submitOneGateRequest function**
  Implement form submissions, adding new table rows dynamically, and triggering automated Rika approvals after 3 seconds.
  ```javascript
  let reqCount = 22;
  function submitOneGateRequest(event) {
    event.preventDefault();
    const reqTypeSelect = document.getElementById('req-type');
    const reqReasonArea = document.getElementById('req-reason');
    const tbody = document.getElementById('one-gate-history-tbody');
    
    if (!reqTypeSelect || !reqReasonArea || !tbody) return;

    const reqId = `REQ-00${reqCount++}`;
    const reqType = reqTypeSelect.value;
    const today = new Date();
    const formattedDate = `${today.getDate()}/${today.getMonth() + 1}/${today.getFullYear()}`;

    // Append new Request row (Pending status)
    const newRow = document.createElement('tr');
    newRow.id = `row-${reqId}`;
    newRow.innerHTML = `
      <td>${reqId}</td>
      <td>${reqType}</td>
      <td>🤖 Rika (Cố vấn)</td>
      <td><span id="status-${reqId}" style="background:rgba(251,191,36,0.15); color:var(--warning); padding:0.15rem 0.4rem; border-radius:4px; font-weight:600;">Đang xử lý</span></td>
      <td>${formattedDate}</td>
    `;
    
    // Insert at the beginning of the table history body
    tbody.insertBefore(newRow, tbody.firstChild);

    // Reset Form Input
    reqReasonArea.value = '';

    // Alert simulation start
    alert(`Đơn ${reqId} đã gửi lên Cổng 1 Cửa. Rika đang duyệt đơn tự động...`);

    // Simulated auto-approval in 3 seconds
    setTimeout(() => {
      const statusSpan = document.getElementById(`status-${reqId}`);
      if (statusSpan) {
        statusSpan.innerText = 'Đã duyệt';
        statusSpan.style.background = 'rgba(52,211,153,0.15)';
        statusSpan.style.color = 'var(--success)';
        alert(`🔔 AI Rika đã tự động phê duyệt đơn ${reqId}!`);
      }
    }, 3000);
  }
  ```

- [ ] **Step 2: Commit one-gate request flow**
  Run: `git add index.html`
  Run: `git commit -m "feat: implement dynamic One-Gate request submission and auto-approvals"`

---

### Task 8: Verification, Cleanup, and Deployment to Vercel

**Files:**
- Create: none (cleanup)
- Deploy: Vercel

- [ ] **Step 1: Clean up temporary extraction folder**
  Remove the `temp_xlsx` directory.
  Run: `Remove-Item -Recurve -Force temp_xlsx`

- [ ] **Step 2: Run verification checks locally**
  Open the file `index.html` in the browser or check in the companion workspace. Confirm the UI switches, chatbot types out outer joins/SQL lessons, streak increments, and requests successfully get approved in 3 seconds.

- [ ] **Step 3: Deploy to Vercel**
  Deploy using the active Vercel configurations.
  Run: `npx vercel --prod --yes`
  Expected output: Production URL link shown in console.

- [ ] **Step 4: Commit changes and verify git status**
  Confirm workspace is clean.
  Run: `git status`
  Expected: No modifications remaining outside specs/plans.
