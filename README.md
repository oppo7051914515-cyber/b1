<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ระบบเช็คชื่อนักเรียนด้วย QR Code + OTP</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts (Sarabun / Inter) -->
    <link href="https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">
    <!-- Font Awesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- SweetAlert2 -->
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <!-- QRCode.js Library -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <!-- Canvas Confetti for Success Animations -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Sarabun', 'sans-serif'],
                    },
                    colors: {
                        brand: {
                            50: '#f0f9ff',
                            100: '#e0f2fe',
                            500: '#0284c7',
                            600: '#0284c7',
                            700: '#0369a1',
                        }
                    }
                }
            }
        }
    </script>
    <style>
        body {
            font-family: 'Sarabun', sans-serif;
            background-color: #f8fafc;
        }
        .otp-letter {
            letter-spacing: 0.35em;
        }
        /* Custom scrollbar */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f1f1;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #94a3b8;
        }
        .glass-card {
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(10px);
        }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen flex flex-col justify-between selection:bg-indigo-500 selection:text-white">

    <!-- Top Navigation / Role Switcher -->
    <header class="bg-slate-900 text-white shadow-lg sticky top-0 z-50 border-b border-slate-800">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 flex items-center justify-between">
            <div class="flex items-center space-x-3">
                <div class="bg-gradient-to-tr from-indigo-600 to-violet-500 p-2.5 rounded-xl text-white font-bold shadow-md shadow-indigo-500/30">
                    <i class="fa-solid fa-qrcode text-xl"></i>
                </div>
                <div>
                    <h1 class="font-extrabold text-lg leading-tight tracking-tight text-white flex items-center gap-2">
                        Smart Check-in
                        <span class="text-[10px] bg-indigo-500/20 text-indigo-300 font-semibold px-2 py-0.5 rounded-md border border-indigo-400/30">v2.5</span>
                    </h1>
                    <p class="text-xs text-slate-400">ระบบเช็คชื่อนักเรียนด้วย QR Code + OTP</p>
                </div>
            </div>

            <!-- Global Role Switcher -->
            <div class="flex items-center bg-slate-800 p-1 rounded-2xl border border-slate-700/80 shadow-inner">
                <button id="btn-role-teacher" onclick="switchRole('teacher')" class="px-3.5 py-1.5 rounded-xl text-xs sm:text-sm font-semibold transition-all duration-200 flex items-center space-x-2 bg-indigo-600 text-white shadow-sm">
                    <i class="fa-solid fa-chalkboard-user text-xs"></i>
                    <span>มุมมองคุณครู</span>
                </button>
                <button id="btn-role-student" onclick="switchRole('student')" class="px-3.5 py-1.5 rounded-xl text-xs sm:text-sm font-semibold transition-all duration-200 flex items-center space-x-2 text-slate-400 hover:text-white">
                    <i class="fa-solid fa-user-graduate text-xs"></i>
                    <span>มุมมองนักเรียน</span>
                </button>
            </div>
        </div>
    </header>

    <!-- Main Content Container -->
    <main class="flex-grow max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-6">

        <!-- ========================================== -->
        <!-- 1. TEACHER VIEW SECTION                    -->
        <!-- ========================================== -->
        <section id="teacher-view" class="space-y-6">
            
            <!-- Navigation Tabs for Teacher Dashboard -->
            <div class="bg-white rounded-2xl p-1.5 shadow-sm border border-slate-200 flex flex-wrap gap-1 sm:gap-2 text-xs sm:text-sm font-semibold">
                <button onclick="switchTeacherTab('tab-dashboard')" id="nav-tab-dashboard" class="teacher-tab-btn flex-1 min-w-[110px] sm:min-w-[130px] px-3.5 py-2.5 rounded-xl flex items-center justify-center space-x-2 bg-indigo-50 text-indigo-700 transition">
                    <i class="fa-solid fa-chart-pie"></i>
                    <span>ภาพรวม</span>
                </button>
                <button onclick="switchTeacherTab('tab-classes')" id="nav-tab-classes" class="teacher-tab-btn flex-1 min-w-[110px] sm:min-w-[130px] px-3.5 py-2.5 rounded-xl flex items-center justify-center space-x-2 text-slate-600 hover:bg-slate-100 transition">
                    <i class="fa-solid fa-school"></i>
                    <span>จัดการชั้นเรียน & วิชา</span>
                </button>
                <button onclick="switchTeacherTab('tab-session')" id="nav-tab-session" class="teacher-tab-btn flex-1 min-w-[110px] sm:min-w-[130px] px-3.5 py-2.5 rounded-xl flex items-center justify-center space-x-2 text-slate-600 hover:bg-slate-100 transition">
                    <i class="fa-solid fa-qrcode"></i>
                    <span>เปิดคาบเรียน & QR</span>
                </button>
                <button onclick="switchTeacherTab('tab-verify')" id="nav-tab-verify" class="teacher-tab-btn flex-1 min-w-[110px] sm:min-w-[130px] px-3.5 py-2.5 rounded-xl flex items-center justify-center space-x-2 text-slate-600 hover:bg-slate-100 transition">
                    <i class="fa-solid fa-key"></i>
                    <span>ยืนยันรหัส OTP</span>
                </button>
                <button onclick="switchTeacherTab('tab-students')" id="nav-tab-students" class="teacher-tab-btn flex-1 min-w-[110px] sm:min-w-[130px] px-3.5 py-2.5 rounded-xl flex items-center justify-center space-x-2 text-slate-600 hover:bg-slate-100 transition">
                    <i class="fa-solid fa-users"></i>
                    <span>จัดการนักเรียน</span>
                </button>
                <button onclick="switchTeacherTab('tab-reports')" id="nav-tab-reports" class="teacher-tab-btn flex-1 min-w-[110px] sm:min-w-[130px] px-3.5 py-2.5 rounded-xl flex items-center justify-center space-x-2 text-slate-600 hover:bg-slate-100 transition">
                    <i class="fa-solid fa-file-lines"></i>
                    <span>รายงาน & สถิติ</span>
                </button>
            </div>

            <!-- TAB 1: DASHBOARD OVERVIEW -->
            <div id="tab-dashboard" class="teacher-tab-content space-y-6">
                <!-- Summary Stats Cards -->
                <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 flex items-center space-x-4">
                        <div class="w-12 h-12 rounded-2xl bg-indigo-50 text-indigo-600 flex items-center justify-center text-xl shadow-inner">
                            <i class="fa-solid fa-users"></i>
                        </div>
                        <div>
                            <p class="text-xs font-semibold text-slate-400 uppercase tracking-wider">นักเรียนทั้งหมด</p>
                            <h3 class="text-2xl font-black text-slate-800" id="stat-total-students">0</h3>
                        </div>
                    </div>

                    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 flex items-center space-x-4">
                        <div class="w-12 h-12 rounded-2xl bg-emerald-50 text-emerald-600 flex items-center justify-center text-xl shadow-inner">
                            <i class="fa-solid fa-circle-check"></i>
                        </div>
                        <div>
                            <p class="text-xs font-semibold text-slate-400 uppercase tracking-wider">เข้าเรียนแล้ว (🟢)</p>
                            <h3 class="text-2xl font-black text-emerald-600" id="stat-present-today">0</h3>
                        </div>
                    </div>

                    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 flex items-center space-x-4">
                        <div class="w-12 h-12 rounded-2xl bg-amber-50 text-amber-600 flex items-center justify-center text-xl shadow-inner">
                            <i class="fa-solid fa-clock"></i>
                        </div>
                        <div>
                            <p class="text-xs font-semibold text-slate-400 uppercase tracking-wider">รอครูยืนยัน (🟡)</p>
                            <h3 class="text-2xl font-black text-amber-600" id="stat-pending-today">0</h3>
                        </div>
                    </div>

                    <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200 flex items-center space-x-4">
                        <div class="w-12 h-12 rounded-2xl bg-rose-50 text-rose-600 flex items-center justify-center text-xl shadow-inner">
                            <i class="fa-solid fa-user-xmark"></i>
                        </div>
                        <div>
                            <p class="text-xs font-semibold text-slate-400 uppercase tracking-wider">ยังไม่เช็คชื่อ (🔴)</p>
                            <h3 class="text-2xl font-black text-rose-600" id="stat-absent-today">0</h3>
                        </div>
                    </div>
                </div>

                <!-- Real-time Attendance Live Dashboard -->
                <div class="bg-white rounded-2xl shadow-sm border border-slate-200 p-6 space-y-4">
                    <div class="flex flex-col md:flex-row md:items-center justify-between gap-4 border-b border-slate-100 pb-4">
                        <div>
                            <h2 class="text-lg font-bold text-slate-800 flex items-center gap-2">
                                <i class="fa-solid fa-tower-broadcast text-indigo-600 animate-pulse"></i>
                                รายชื่อการเช็คชื่อเรียลไทม์ (คาบปัจจุบัน)
                            </h2>
                            <p class="text-xs text-slate-500 mt-1" id="dash-active-session-title">กำลังโหลดข้อมูลคาบเรียน...</p>
                        </div>
                        <div class="flex flex-wrap items-center gap-2">
                            <button onclick="exportToCSV()" class="px-3.5 py-2 rounded-xl bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-semibold flex items-center gap-2 transition shadow-sm">
                                <i class="fa-solid fa-file-excel"></i>
                                <span>ส่งออก CSV</span>
                            </button>
                            <button onclick="resetDataToDefault()" class="px-3.5 py-2 rounded-xl bg-slate-100 hover:bg-slate-200 text-slate-600 text-xs font-semibold flex items-center gap-1.5 transition">
                                <i class="fa-solid fa-rotate-left"></i>
                                <span>รีเซ็ตตัวอย่าง</span>
                            </button>
                        </div>
                    </div>

                    <!-- Attendance Table -->
                    <div class="overflow-x-auto">
                        <table class="w-full text-left text-sm border-collapse">
                            <thead>
                                <tr class="bg-slate-50 text-slate-500 font-semibold border-b border-slate-200">
                                    <th class="p-3">เลขที่</th>
                                    <th class="p-3">รหัสนักเรียน</th>
                                    <th class="p-3">ชื่อ-นามสกุล</th>
                                    <th class="p-3 text-center">รหัส OTP</th>
                                    <th class="p-3 text-center">เวลาสแกน / ยืนยัน</th>
                                    <th class="p-3 text-center">สถานะ</th>
                                    <th class="p-3 text-center">การจัดการ</th>
                                </tr>
                            </thead>
                            <tbody id="table-dashboard-attendance" class="divide-y divide-slate-100">
                                <!-- Dynamic Render -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>

            <!-- TAB 2: CLASS & SUBJECT MANAGEMENT (Create & Delete Freely) -->
            <div id="tab-classes" class="teacher-tab-content hidden space-y-6">
                <div class="grid grid-cols-1 lg:grid-cols-12 gap-6">
                    
                    <!-- Class Management (Add/Delete Freely) -->
                    <div class="lg:col-span-7 bg-white rounded-2xl p-6 shadow-sm border border-slate-200 space-y-5">
                        <div class="flex items-center justify-between border-b border-slate-100 pb-3">
                            <div>
                                <h2 class="text-base font-bold text-slate-800 flex items-center gap-2">
                                    <i class="fa-solid fa-school text-indigo-600"></i>
                                    จัดการชั้นเรียน/ห้องเรียน (Class Management)
                                </h2>
                                <p class="text-xs text-slate-500 mt-0.5">คุณครูสามารถสร้างชั้นเรียนใหม่ หรือ ลบชั้นเรียนเดิมได้ตามต้องการ</p>
                            </div>
                            <span id="badge-total-classes" class="px-3 py-1 text-xs font-bold bg-indigo-50 text-indigo-700 rounded-full border border-indigo-100">
                                0 ชั้นเรียน
                            </span>
                        </div>

                        <!-- Add New Class Form -->
                        <form onsubmit="handleCreateNewClass(event)" class="bg-gradient-to-r from-indigo-50/70 to-blue-50/70 p-4 rounded-2xl border border-indigo-100 space-y-3">
                            <p class="text-xs font-bold text-indigo-900 flex items-center gap-1.5">
                                <i class="fa-solid fa-plus-circle text-indigo-600"></i>
                                สร้างชั้นเรียนใหม่ (Add Class)
                            </p>
                            <div class="grid grid-cols-1 sm:grid-cols-2 gap-3">
                                <div>
                                    <label class="block text-[11px] font-semibold text-slate-600 mb-1">ชื่อชั้นเรียน / ห้องเรียน *</label>
                                    <input type="text" id="input-class-name" placeholder="เช่น ม.1/1, ม.4/3 (วิทย์-คณิต), ปวช.1" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 text-xs focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-white shadow-sm">
                                </div>
                                <div>
                                    <label class="block text-[11px] font-semibold text-slate-600 mb-1">ภาคเรียน / ปีการศึกษา</label>
                                    <input type="text" id="input-class-term" placeholder="เช่น 1/2569" class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 text-xs focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-white shadow-sm">
                                </div>
                            </div>
                            <div class="flex justify-end pt-1">
                                <button type="submit" class="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl text-xs font-bold shadow-md shadow-indigo-500/20 transition flex items-center gap-1.5">
                                    <i class="fa-solid fa-plus"></i>
                                    <span>บันทึกสร้างชั้นเรียน</span>
                                </button>
                            </div>
                        </form>

                        <!-- Classes List Table -->
                        <div class="overflow-x-auto">
                            <table class="w-full text-left text-xs border-collapse">
                                <thead>
                                    <tr class="bg-slate-50 text-slate-500 font-semibold border-b border-slate-200">
                                        <th class="p-3">ชื่อชั้นเรียน</th>
                                        <th class="p-3">ภาคเรียน</th>
                                        <th class="p-3 text-center">จำนวนนักเรียน</th>
                                        <th class="p-3 text-center">จัดการ</th>
                                    </tr>
                                </thead>
                                <tbody id="table-classes-list" class="divide-y divide-slate-100">
                                    <!-- Dynamic Render Classes -->
                                </tbody>
                            </table>
                        </div>
                    </div>

                    <!-- Subject Management -->
                    <div class="lg:col-span-5 bg-white rounded-2xl p-6 shadow-sm border border-slate-200 space-y-4">
                        <div class="flex items-center justify-between border-b border-slate-100 pb-3">
                            <div>
                                <h2 class="text-base font-bold text-slate-800 flex items-center gap-2">
                                    <i class="fa-solid fa-book text-indigo-600"></i>
                                    จัดการรายวิชา (Subject Management)
                                </h2>
                                <p class="text-xs text-slate-500 mt-0.5">เพิ่มหรือลบรายวิชาสำหรับการเปิดคาบเรียน</p>
                            </div>
                        </div>

                        <form onsubmit="handleAddSubject(event)" class="flex gap-2">
                            <input type="text" id="input-new-subject" placeholder="เช่น คณิตศาสตร์เพิ่มเติม, ฟิสิกส์" required class="flex-1 px-3.5 py-2.5 rounded-xl border border-slate-200 text-xs focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <button type="submit" class="px-4 py-2.5 bg-indigo-600 text-white rounded-xl text-xs font-bold hover:bg-indigo-700 transition flex items-center gap-1 shadow-md shadow-indigo-500/20">
                                <i class="fa-solid fa-plus"></i>
                                <span>เพิ่มวิชา</span>
                            </button>
                        </form>

                        <ul id="list-subjects" class="divide-y divide-slate-100 max-h-80 overflow-y-auto pr-1">
                            <!-- Dynamic Subject List -->
                        </ul>
                    </div>
                </div>
            </div>

            <!-- TAB 3: OPEN SESSION & QR CODE -->
            <div id="tab-session" class="teacher-tab-content hidden space-y-6">
                <div class="grid grid-cols-1 lg:grid-cols-12 gap-6">
                    <!-- Session Setup Control -->
                    <div class="lg:col-span-5 bg-white rounded-2xl p-6 shadow-sm border border-slate-200 space-y-5">
                        <h2 class="text-lg font-bold text-slate-800 border-b border-slate-100 pb-3 flex items-center gap-2">
                            <i class="fa-solid fa-sliders text-indigo-600"></i>
                            ตั้งค่าและเปิดคาบเรียน
                        </h2>

                        <form id="form-create-session" onsubmit="handleCreateSession(event)" class="space-y-4">
                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">เลือกชั้นเรียน / ห้องเรียน *</label>
                                <select id="select-session-class" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm focus:ring-2 focus:ring-indigo-500 focus:outline-none bg-white">
                                    <!-- Dynamic Options -->
                                </select>
                            </div>

                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">เลือกรายวิชา *</label>
                                <select id="select-session-subject" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm focus:ring-2 focus:ring-indigo-500 focus:outline-none bg-white">
                                    <!-- Dynamic Options -->
                                </select>
                            </div>

                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">คาบเรียน / หัวข้อบทเรียน</label>
                                <input type="text" id="input-session-name" placeholder="เช่น คาบที่ 1-2 บทที่ 3 เรื่อง แคลคูลัส" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm focus:ring-2 focus:ring-indigo-500 focus:outline-none">
                            </div>

                            <div>
                                <label class="block text-xs font-semibold text-slate-600 mb-1">ระยะเวลาหมดอายุ QR Code</label>
                                <select id="select-session-expire" class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm focus:ring-2 focus:ring-indigo-500 focus:outline-none bg-white">
                                    <option value="3">3 นาที</option>
                                    <option value="5" selected>5 นาที</option>
                                    <option value="10">10 นาที</option>
                                    <option value="15">15 นาที</option>
                                </select>
                            </div>

                            <button type="submit" class="w-full py-3 bg-indigo-600 hover:bg-indigo-700 text-white font-bold rounded-xl text-sm transition shadow-lg shadow-indigo-500/25 flex items-center justify-center space-x-2">
                                <i class="fa-solid fa-play"></i>
                                <span>เปิดคาบเรียนใหม่ & สร้าง QR Code</span>
                            </button>
                        </form>
                    </div>

                    <!-- Dynamic QR Code Display Container -->
                    <div class="lg:col-span-7 bg-white rounded-2xl p-6 shadow-sm border border-slate-200 flex flex-col items-center justify-center text-center space-y-4 min-h-[420px]">
                        <div id="qr-active-container" class="space-y-4 w-full flex flex-col items-center">
                            <span class="inline-flex items-center px-3 py-1 rounded-full text-xs font-bold bg-emerald-100 text-emerald-800">
                                <span class="w-2 h-2 mr-1.5 bg-emerald-500 rounded-full animate-ping"></span>
                                คาบเรียนกำลังดำเนินอยู่
                            </span>

                            <h3 id="qr-session-title" class="text-xl font-extrabold text-slate-800">ม.4/1 - วิชาคณิตศาสตร์</h3>
                            <p id="qr-session-period" class="text-xs text-slate-500">คาบเรียน: คาบที่ 1-2 เรื่อง แคลคูลัส</p>

                            <!-- QR Code Render Box -->
                            <div class="p-4 bg-white rounded-3xl border-4 border-indigo-50 shadow-inner flex justify-center items-center">
                                <div id="qrcode-element" class="p-2 bg-white rounded-xl"></div>
                            </div>

                            <!-- Countdown Timer -->
                            <div class="space-y-1">
                                <p class="text-xs text-slate-400 font-medium">QR Code หมดอายุในอีก</p>
                                <div id="qr-timer-display" class="text-3xl font-black text-amber-600 tracking-wider">05:00</div>
                            </div>

                            <div class="flex flex-wrap items-center justify-center gap-3 pt-2">
                                <button onclick="refreshActiveQrcode()" class="px-4 py-2.5 bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs font-bold rounded-xl transition flex items-center gap-2">
                                    <i class="fa-solid fa-arrows-rotate"></i>
                                    <span>รีเฟรชรหัส QR</span>
                                </button>
                                <button onclick="simulateStudentScan()" class="px-4 py-2.5 bg-indigo-50 hover:bg-indigo-100 text-indigo-700 text-xs font-bold rounded-xl transition flex items-center gap-2">
                                    <i class="fa-solid fa-mobile-screen-button"></i>
                                    <span>ทดลองสแกน QR (ฝั่งนักเรียน)</span>
                                </button>
                            </div>
                        </div>

                        <!-- Placeholder when no session active -->
                        <div id="qr-empty-container" class="hidden flex-col items-center justify-center space-y-3 py-12">
                            <div class="w-16 h-16 bg-slate-100 text-slate-400 rounded-full flex items-center justify-center text-2xl">
                                <i class="fa-solid fa-qrcode"></i>
                            </div>
                            <p class="text-slate-500 font-medium text-sm">ยังไม่มีการเปิดคาบเรียนในขณะนี้</p>
                            <p class="text-xs text-slate-400 max-w-xs">กรุณาเลือกชั้นเรียน วิชา และกดยืนยันเปิดคาบเรียนด้านซ้ายเพื่อรับ QR Code</p>
                        </div>
                    </div>
                </div>
            </div>

            <!-- TAB 4: TEACHER OTP VERIFICATION -->
            <div id="tab-verify" class="teacher-tab-content hidden space-y-6">
                <div class="max-w-2xl mx-auto bg-white rounded-2xl p-6 sm:p-8 shadow-sm border border-slate-200 space-y-6">
                    <div class="text-center space-y-2">
                        <div class="w-14 h-14 bg-indigo-100 text-indigo-600 rounded-2xl mx-auto flex items-center justify-center text-2xl shadow-inner">
                            <i class="fa-solid fa-key"></i>
                        </div>
                        <h2 class="text-xl font-extrabold text-slate-800">ยืนยันรหัสเข้าเรียน (OTP Verification)</h2>
                        <p class="text-xs text-slate-500">กรอกรหัส OTP 6 หลักที่นักเรียนนำมารายงาน เพื่อเปลี่ยนสถานะเป็น 🟢 เข้าเรียนแล้ว</p>
                    </div>

                    <!-- Input Code Form -->
                    <form onsubmit="handleManualVerifyOTP(event)" class="space-y-4">
                        <div class="flex justify-center">
                            <input type="text" id="input-verify-otp" maxlength="6" pattern="[0-9]{6}" placeholder="000000" required autofocus autocomplete="off" class="w-64 text-center text-3xl font-mono font-black tracking-widest py-3 border-2 border-indigo-300 rounded-2xl focus:ring-4 focus:ring-indigo-100 focus:border-indigo-600 focus:outline-none uppercase bg-indigo-50/40 text-indigo-900 shadow-inner">
                        </div>
                        <div class="flex justify-center">
                            <button type="submit" class="px-8 py-3 bg-indigo-600 hover:bg-indigo-700 text-white font-bold rounded-xl text-sm transition shadow-lg shadow-indigo-500/25 flex items-center space-x-2">
                                <i class="fa-solid fa-check-double"></i>
                                <span>ยืนยันเข้าเรียน</span>
                            </button>
                        </div>
                    </form>

                    <!-- Pending Approval List -->
                    <div class="border-t border-slate-100 pt-5 space-y-3">
                        <h3 class="text-sm font-bold text-slate-700 flex items-center justify-between">
                            <span>รายการที่นักเรียนสแกนแล้ว (รอคุณครูยืนยัน 🟡)</span>
                            <span id="pending-count-badge" class="px-2.5 py-0.5 text-xs bg-amber-100 text-amber-700 font-bold rounded-full">0 รายการ</span>
                        </h3>

                        <div id="pending-otp-list" class="space-y-2 max-h-60 overflow-y-auto pr-1">
                            <!-- Dynamic Render Pending Cards -->
                        </div>
                    </div>
                </div>
            </div>

            <!-- TAB 5: STUDENT MANAGEMENT -->
            <div id="tab-students" class="teacher-tab-content hidden space-y-6">
                <div class="bg-white rounded-2xl p-6 shadow-sm border border-slate-200 space-y-5">
                    <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 border-b border-slate-100 pb-4">
                        <div>
                            <h2 class="text-lg font-bold text-slate-800 flex items-center gap-2">
                                <i class="fa-solid fa-users-gear text-indigo-600"></i>
                                จัดการรายชื่อนักเรียน
                            </h2>
                            <p class="text-xs text-slate-500 mt-1">เพิ่ม ปรับเปลี่ยน หรือกำหนดชั้นเรียนของนักเรียน</p>
                        </div>

                        <div class="flex flex-wrap items-center gap-2">
                            <button onclick="importMockExcelStudents()" class="px-3 py-2 bg-emerald-50 hover:bg-emerald-100 text-emerald-700 text-xs font-bold rounded-xl transition flex items-center gap-1.5 border border-emerald-200">
                                <i class="fa-solid fa-file-import"></i>
                                <span>นำเข้าข้อมูลจำลอง</span>
                            </button>
                            <button onclick="openAddStudentModal()" class="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white text-xs font-bold rounded-xl transition flex items-center gap-2 shadow-md shadow-indigo-500/20">
                                <i class="fa-solid fa-plus"></i>
                                <span>เพิ่มนักเรียนใหม่</span>
                            </button>
                        </div>
                    </div>

                    <!-- Students Filter -->
                    <div class="flex flex-wrap items-center justify-between gap-3">
                        <div class="flex items-center gap-2">
                            <label class="text-xs font-bold text-slate-600">กรองตามชั้นเรียน:</label>
                            <select id="filter-student-class" onchange="renderStudentManagementTable()" class="px-3.5 py-1.5 rounded-xl border border-slate-200 text-xs font-semibold focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-white">
                                <option value="ALL">ทุกชั้นเรียน</option>
                                <!-- Dynamic options -->
                            </select>
                        </div>
                        <span id="student-count-badge" class="text-xs font-bold text-slate-500"></span>
                    </div>

                    <!-- Student List Table -->
                    <div class="overflow-x-auto">
                        <table class="w-full text-left text-sm border-collapse">
                            <thead>
                                <tr class="bg-slate-50 text-slate-500 font-semibold border-b border-slate-200">
                                    <th class="p-3">เลขที่</th>
                                    <th class="p-3">รหัสนักเรียน</th>
                                    <th class="p-3">ชื่อ-นามสกุล</th>
                                    <th class="p-3">ชั้นเรียน</th>
                                    <th class="p-3 text-center">การจัดการ</th>
                                </tr>
                            </thead>
                            <tbody id="table-students-list" class="divide-y divide-slate-100">
                                <!-- Dynamic Render Students -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>

            <!-- TAB 6: REPORTS & STATISTICS -->
            <div id="tab-reports" class="teacher-tab-content hidden space-y-6">
                <div class="bg-white rounded-2xl p-6 shadow-sm border border-slate-200 space-y-6">
                    <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 border-b border-slate-100 pb-4">
                        <div>
                            <h2 class="text-lg font-bold text-slate-800 flex items-center gap-2">
                                <i class="fa-solid fa-chart-column text-indigo-600"></i>
                                สรุปรายงานและสถิติการเข้าเรียน
                            </h2>
                            <p class="text-xs text-slate-500 mt-0.5">ภาพรวมสถิติการเช็คชื่อเข้าเรียนของทุกชั้นเรียน</p>
                        </div>
                        <button onclick="exportToCSV()" class="px-4 py-2.5 bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-bold rounded-xl transition flex items-center gap-2 shadow-md shadow-emerald-500/20">
                            <i class="fa-solid fa-file-excel"></i>
                            <span>ส่งออกรายงานเป็น CSV</span>
                        </button>
                    </div>

                    <!-- Report Cards Summary -->
                    <div id="report-summary-cards" class="grid grid-cols-1 sm:grid-cols-3 gap-4">
                        <!-- Dynamic Stats Cards for Report -->
                    </div>

                    <!-- Filter Controls -->
                    <div class="flex flex-wrap items-center gap-3 bg-slate-50 p-4 rounded-xl border border-slate-200">
                        <label class="text-xs font-bold text-slate-700">เลือกชั้นเรียน:</label>
                        <select id="report-filter-class" onchange="renderReportTable()" class="px-3.5 py-2 rounded-xl border border-slate-200 text-xs font-semibold focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-white">
                            <!-- Dynamic Class Options -->
                        </select>
                    </div>

                    <!-- Detailed Attendance Log Table -->
                    <div class="overflow-x-auto">
                        <table class="w-full text-left text-sm border-collapse">
                            <thead>
                                <tr class="bg-slate-100 text-slate-600 font-bold border-b border-slate-200">
                                    <th class="p-3">เลขที่</th>
                                    <th class="p-3">รหัส</th>
                                    <th class="p-3">ชื่อ-นามสกุล</th>
                                    <th class="p-3">ชั้นเรียน</th>
                                    <th class="p-3 text-center">สถานะ</th>
                                    <th class="p-3 text-center">เวลาสแกน</th>
                                    <th class="p-3 text-center">เวลายืนยัน</th>
                                </tr>
                            </thead>
                            <tbody id="table-report-attendance" class="divide-y divide-slate-100">
                                <!-- Dynamic Render Report -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>

        </section>

        <!-- ========================================== -->
        <!-- 2. STUDENT VIEW SECTION (Mobile App UI)    -->
        <!-- ========================================== -->
        <section id="student-view" class="hidden max-w-md mx-auto space-y-4">
            
            <!-- Mobile Container Frame -->
            <div class="bg-white rounded-3xl shadow-xl border border-slate-200 overflow-hidden">
                <!-- Mobile Top Header -->
                <div class="bg-gradient-to-r from-indigo-700 via-indigo-800 to-slate-900 text-white p-6 text-center space-y-2 shadow-md">
                    <div class="inline-flex p-3 bg-white/10 rounded-2xl backdrop-blur-md mb-1 shadow-inner">
                        <i class="fa-solid fa-id-badge text-3xl text-indigo-200"></i>
                    </div>
                    <h2 class="text-xl font-black tracking-tight">เช็คชื่อเข้าเรียน (Student)</h2>
                    <p class="text-xs text-indigo-200">สแกน QR Code แล้วนำรหัส OTP 6 หลักแจ้งคุณครู</p>
                </div>

                <!-- Simulation Student Switcher Selector -->
                <div class="bg-slate-100 p-3 border-b border-slate-200 flex items-center justify-between text-xs">
                    <span class="text-slate-600 font-bold"><i class="fa-solid fa-user-circle mr-1 text-indigo-600"></i>เลือกตัวตนนักเรียน:</span>
                    <select id="select-student-identity" onchange="handleStudentIdentityChange()" class="bg-white border border-slate-300 rounded-xl px-2.5 py-1 text-xs text-slate-800 font-bold focus:outline-none focus:ring-2 focus:ring-indigo-500 shadow-sm">
                        <!-- Dynamic Student Options -->
                    </select>
                </div>

                <!-- Main Content Body -->
                <div id="student-content-container" class="p-6 space-y-6">
                    <!-- Dynamic rendering of student status -->
                </div>
            </div>

            <!-- Helpful Notice -->
            <p class="text-center text-xs text-slate-400">
                <i class="fa-solid fa-shield-halved mr-1 text-indigo-500"></i>
                หน้านี้จำลองหน้าจอมือถือนักเรียนเพื่อความสะดวกในการทดสอบระบบ
            </p>
        </section>

    </main>

    <!-- Footer -->
    <footer class="bg-white border-t border-slate-200 py-4 mt-8">
        <div class="max-w-7xl mx-auto px-4 text-center text-xs text-slate-400 font-medium">
            &copy; 2026 Smart Attendance Systemด้วย QR Code + OTP | ระบบจัดการชั้นเรียนและเช็คชื่ออัจฉริยะ
        </div>
    </footer>

    <!-- Modal: Add Student -->
    <div id="modal-add-student" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm z-50 flex items-center justify-center hidden p-4">
        <div class="bg-white rounded-3xl max-w-md w-full p-6 space-y-4 shadow-2xl border border-slate-100">
            <div class="flex items-center justify-between border-b border-slate-100 pb-3">
                <h3 class="font-bold text-slate-800 text-lg flex items-center gap-2">
                    <i class="fa-solid fa-user-plus text-indigo-600"></i>
                    เพิ่มนักเรียนใหม่
                </h3>
                <button onclick="closeAddStudentModal()" class="text-slate-400 hover:text-slate-600 p-1">
                    <i class="fa-solid fa-xmark text-lg"></i>
                </button>
            </div>

            <form onsubmit="handleSaveStudent(event)" class="space-y-4">
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">รหัสนักเรียน *</label>
                    <input type="text" id="input-student-code" required placeholder="เช่น 10001" class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500">
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">ชื่อ-นามสกุล *</label>
                    <input type="text" id="input-student-name" required placeholder="เช่น นายสมชาย ใจดี" class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500">
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">เลขที่ *</label>
                    <input type="number" id="input-student-number" min="1" required placeholder="เช่น 1" class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500">
                </div>
                <div>
                    <label class="block text-xs font-semibold text-slate-600 mb-1">เลือกชั้นเรียน *</label>
                    <select id="select-student-class" required class="w-full px-3.5 py-2.5 rounded-xl border border-slate-200 text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500 bg-white">
                        <!-- Dynamic Options -->
                    </select>
                </div>

                <div class="flex justify-end space-x-2 pt-2">
                    <button type="button" onclick="closeAddStudentModal()" class="px-4 py-2.5 rounded-xl text-slate-600 hover:bg-slate-100 text-xs font-bold transition">ยกเลิก</button>
                    <button type="submit" class="px-5 py-2.5 bg-indigo-600 text-white rounded-xl text-xs font-bold hover:bg-indigo-700 transition shadow-md shadow-indigo-500/20">บันทึกข้อมูล</button>
                </div>
            </form>
        </div>
    </div>

    <script>
        // Default Core Data Structures
        const DEFAULT_CLASSES = [
            { id: "c1", name: "ม.4/1", term: "1/2569" },
            { id: "c2", name: "ม.4/2", term: "1/2569" },
            { id: "c3", name: "ม.5/1 (วิทย์-คณิต)", term: "1/2569" }
        ];

        const DEFAULT_SUBJECTS = [
            { id: "s1", name: "คณิตศาสตร์เพิ่มเติม" },
            { id: "s2", name: "วิทยาศาสตร์และเทคโนโลยี" },
            { id: "s3", name: "ภาษาอังกฤษพื้นฐาน" }
        ];

        const DEFAULT_STUDENTS = [
            { id: "st1", code: "10001", name: "นายสมชาย สุขใจ", number: 1, classId: "c1" },
            { id: "st2", code: "10002", name: "นางสาวมณี รัตนะ", number: 2, classId: "c1" },
            { id: "st3", code: "10003", name: "นายปิติ มีสุข", number: 3, classId: "c1" },
            { id: "st4", code: "10004", name: "นางสาวชูใจ บุญเลิศ", number: 4, classId: "c1" },
            { id: "st5", code: "10005", name: "นายวีระ ชนะศึก", number: 5, classId: "c1" },
            { id: "st6", code: "20001", name: "นายก้องเกียรติ รักดี", number: 1, classId: "c2" },
            { id: "st7", code: "20002", name: "นางสาวศิริพร งามยิ่ง", number: 2, classId: "c2" }
        ];

        // Global Application State Container
        let appState = {
            classes: [],
            subjects: [],
            students: [],
            activeSession: null, // { id, classId, subjectId, sessionName, expireMinutes, startTime, endTime, qrToken }
            attendance: [], // { studentId, sessionId, status: 'RED'|'YELLOW'|'GREEN', otp, scanTime, verifyTime }
            currentRole: 'teacher',
            selectedStudentId: null
        };

        let countdownTimer = null;

        // On Page Load Initialization
        window.onload = function () {
            loadStateFromStorage();
            if (!appState.classes || appState.classes.length === 0) {
                resetDataToDefault(false);
            }
            
            // Set default selected student identity for mobile simulator
            if (appState.students.length > 0 && !appState.selectedStudentId) {
                appState.selectedStudentId = appState.students[0].id;
            }

            renderAllViews();
            startAutoRefreshTimer();
        };

        // Storage Persistence Functions
        function saveStateToStorage() {
            localStorage.setItem('smart_checkin_app_state_v2', JSON.stringify(appState));
        }

        function loadStateFromStorage() {
            const data = localStorage.getItem('smart_checkin_app_state_v2');
            if (data) {
                try {
                    appState = JSON.parse(data);
                } catch (e) {
                    console.error('Failed to parse appState', e);
                }
            }
        }

        function resetDataToDefault(confirm = true) {
            if (confirm && !window.confirm('คุณต้องการรีเซ็ตข้อมูลทั้งหมดกลับเป็นค่าเริ่มต้นใช่หรือไม่?')) return;
            
            const now = Date.now();
            const defaultSession = {
                id: "sess_101",
                classId: "c1",
                subjectId: "s1",
                sessionName: "คาบที่ 1-2 แคลคูลัสเบื้องต้น",
                expireMinutes: 5,
                startTime: now,
                endTime: now + (5 * 60 * 1000),
                qrToken: "QR_" + Math.random().toString(36).substr(2, 6).toUpperCase()
            };

            appState = {
                classes: [...DEFAULT_CLASSES],
                subjects: [...DEFAULT_SUBJECTS],
                students: [...DEFAULT_STUDENTS],
                activeSession: defaultSession,
                attendance: [
                    { studentId: "st1", sessionId: "sess_101", status: 'GREEN', otp: '582910', scanTime: new Date(now - 1000*60*4).toLocaleTimeString('th-TH'), verifyTime: new Date(now - 1000*60*3).toLocaleTimeString('th-TH') },
                    { studentId: "st2", sessionId: "sess_101", status: 'YELLOW', otp: '149203', scanTime: new Date(now - 1000*60*2).toLocaleTimeString('th-TH'), verifyTime: null },
                    { studentId: "st3", sessionId: "sess_101", status: 'RED', otp: null, scanTime: null, verifyTime: null },
                    { studentId: "st4", sessionId: "sess_101", status: 'RED', otp: null, scanTime: null, verifyTime: null },
                    { studentId: "st5", sessionId: "sess_101", status: 'RED', otp: null, scanTime: null, verifyTime: null }
                ],
                currentRole: appState.currentRole || 'teacher',
                selectedStudentId: "st2"
            };

            saveStateToStorage();
            renderAllViews();
            
            if (confirm) {
                Swal.fire({
                    icon: 'success',
                    title: 'รีเซ็ตข้อมูลสำเร็จ',
                    text: 'โหลดข้อมูลตัวอย่างสำหรับทดลองใช้งานเรียบร้อยแล้ว',
                    timer: 1500,
                    showConfirmButton: false
                });
            }
        }

        // ==========================================
        // CLASS MANAGEMENT LOGIC (Add & Delete Freely)
        // ==========================================

        function handleCreateNewClass(e) {
            e.preventDefault();
            const nameInput = document.getElementById('input-class-name');
            const termInput = document.getElementById('input-class-term');

            const className = nameInput.value.trim();
            const classTerm = termInput.value.trim() || "1/2569";

            if (!className) {
                Swal.fire('ข้อผิดพลาด', 'กรุณาระบุชื่อชั้นเรียน/ห้องเรียน', 'warning');
                return;
            }

            // Check duplicate class name (case-insensitive)
            const isDuplicate = appState.classes.some(c => c.name.toLowerCase() === className.toLowerCase());
            if (isDuplicate) {
                Swal.fire('ข้อผิดพลาด', `มีชั้นเรียนชื่อ "${className}" ในระบบแล้ว`, 'warning');
                return;
            }

            const newClass = {
                id: "class_" + Date.now(),
                name: className,
                term: classTerm
            };

            appState.classes.push(newClass);
            saveStateToStorage();

            nameInput.value = '';
            termInput.value = '';

            renderAllViews();

            Swal.fire({
                icon: 'success',
                title: 'สร้างชั้นเรียนสำเร็จ',
                text: `ชั้นเรียน "${className}" ถูกเพิ่มเข้าระบบแล้ว`,
                timer: 1500,
                showConfirmButton: false
            });
        }

        function handleDeleteClass(classId) {
            const targetClass = appState.classes.find(c => c.id === classId);
            if (!targetClass) return;

            const studentCount = appState.students.filter(s => s.classId === classId).length;

            Swal.fire({
                title: `ยืนยันการลบชั้นเรียน ${targetClass.name}?`,
                html: studentCount > 0 
                    ? `<span class="text-rose-600 font-bold">เตือน:</span> ชั้นเรียนนี้มีนักเรียนอยู่ <b class="text-indigo-600">${studentCount} คน</b><br>หากลบชั้นเรียน ข้อมูลนักเรียนประจำชั้นเรียนนี้จะถูกลบออกไปด้วย`
                    : "คุณสามารถสร้างใหม่เมื่อไหร่ก็ได้ที่ต้องการ",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#e11d48',
                cancelButtonColor: '#64748b',
                confirmButtonText: '<i class="fa-solid fa-trash mr-1"></i> ยืนยันลบชั้นเรียน',
                cancelButtonText: 'ยกเลิก'
            }).then((result) => {
                if (result.isConfirmed) {
                    // Delete all students in this class
                    appState.students = appState.students.filter(s => s.classId !== classId);
                    
                    // Delete the class
                    appState.classes = appState.classes.filter(c => c.id !== classId);

                    // If active session belongs to this deleted class, clear active session
                    if (appState.activeSession && appState.activeSession.classId === classId) {
                        appState.activeSession = null;
                    }

                    saveStateToStorage();
                    renderAllViews();

                    Swal.fire({
                        icon: 'success',
                        title: 'ลบชั้นเรียนเรียบร้อยแล้ว',
                        timer: 1500,
                        showConfirmButton: false
                    });
                }
            });
        }

        // ==========================================
        // UI RENDERERS & SYNCHRONIZATION
        // ==========================================

        function renderAllViews() {
            renderTeacherSelects();
            renderClassesAndSubjectsLists();
            renderDashboardStats();
            renderDashboardAttendanceTable();
            renderPendingOTPList();
            renderStudentManagementTable();
            renderReportTable();
            renderActiveSessionQR();
            renderStudentIdentitySelect();
            renderStudentView();
        }

        function renderTeacherSelects() {
            const sessionClassSelect = document.getElementById('select-session-class');
            const sessionSubjectSelect = document.getElementById('select-session-subject');
            const studentClassSelect = document.getElementById('select-student-class');
            const filterClassSelect = document.getElementById('filter-student-class');
            const reportFilterClassSelect = document.getElementById('report-filter-class');

            const classOptionsHTML = appState.classes.length > 0 
                ? appState.classes.map(c => `<option value="${c.id}">${c.name} (${c.term || '1/2569'})</option>`).join('')
                : `<option value="">-- กรุณาสร้างชั้นเรียนก่อน --</option>`;

            const subjectOptionsHTML = appState.subjects.length > 0 
                ? appState.subjects.map(s => `<option value="${s.id}">${s.name}</option>`).join('')
                : `<option value="">-- กรุณาเพิ่มวิชาก่อน --</option>`;

            if (sessionClassSelect) sessionClassSelect.innerHTML = classOptionsHTML;
            if (studentClassSelect) studentClassSelect.innerHTML = classOptionsHTML;
            if (sessionSubjectSelect) sessionSubjectSelect.innerHTML = subjectOptionsHTML;
            
            if (filterClassSelect) {
                filterClassSelect.innerHTML = '<option value="ALL">ทุกชั้นเรียน</option>' + classOptionsHTML;
            }
            if (reportFilterClassSelect) {
                reportFilterClassSelect.innerHTML = classOptionsHTML;
            }
        }

        function renderClassesAndSubjectsLists() {
            const classTableBody = document.getElementById('table-classes-list');
            const classBadge = document.getElementById('badge-total-classes');
            const subjectList = document.getElementById('list-subjects');

            if (classBadge) classBadge.innerText = `${appState.classes.length} ชั้นเรียน`;

            if (classTableBody) {
                if (appState.classes.length === 0) {
                    classTableBody.innerHTML = `<tr><td colspan="4" class="text-center py-6 text-slate-400 font-medium">ยังไม่มีชั้นเรียนในระบบ กรุณาสร้างชั้นเรียนใหม่ด้านบน</td></tr>`;
                } else {
                    classTableBody.innerHTML = appState.classes.map(c => {
                        const count = appState.students.filter(s => s.classId === c.id).length;
                        return `
                            <tr class="hover:bg-slate-50 transition">
                                <td class="p-3 font-bold text-slate-800">
                                    <div class="flex items-center space-x-2">
                                        <div class="w-8 h-8 rounded-xl bg-indigo-50 text-indigo-600 flex items-center justify-center font-bold text-xs shadow-sm">
                                            <i class="fa-solid fa-chalkboard"></i>
                                        </div>
                                        <span>${c.name}</span>
                                    </div>
                                </td>
                                <td class="p-3 text-slate-500 font-semibold">${c.term || '1/2569'}</td>
                                <td class="p-3 text-center">
                                    <span class="px-2.5 py-1 rounded-full text-xs font-bold bg-slate-100 text-slate-700">
                                        <i class="fa-solid fa-user-graduate text-[10px] mr-1 text-slate-400"></i>${count} คน
                                    </span>
                                </td>
                                <td class="p-3 text-center">
                                    <button onclick="handleDeleteClass('${c.id}')" class="px-3 py-1.5 bg-rose-50 hover:bg-rose-100 text-rose-600 rounded-xl text-xs font-bold transition inline-flex items-center gap-1 shadow-sm border border-rose-100">
                                        <i class="fa-solid fa-trash text-[11px]"></i>
                                        <span>ลบ</span>
                                    </button>
                                </td>
                            </tr>
                        `;
                    }).join('');
                }
            }

            if (subjectList) {
                if (appState.subjects.length === 0) {
                    subjectList.innerHTML = `<li class="py-4 text-center text-xs text-slate-400">ยังไม่มีรายวิชา</li>`;
                } else {
                    subjectList.innerHTML = appState.subjects.map(s => `
                        <li class="py-2.5 flex items-center justify-between text-xs">
                            <span class="font-bold text-slate-700"><i class="fa-solid fa-book-bookmark text-indigo-500 mr-2"></i>${s.name}</span>
                            <button onclick="handleDeleteSubject('${s.id}')" class="text-rose-500 hover:text-rose-700 font-semibold text-xs px-2 py-1 flex items-center gap-1 hover:bg-rose-50 rounded-lg transition">
                                <i class="fa-solid fa-trash text-[10px]"></i>ลบ
                            </button>
                        </li>
                    `).join('');
                }
            }
        }

        function handleAddSubject(e) {
            e.preventDefault();
            const input = document.getElementById('input-new-subject');
            const name = input.value.trim();
            if (!name) return;

            appState.subjects.push({
                id: "subj_" + Date.now(),
                name: name
            });

            saveStateToStorage();
            input.value = '';
            renderAllViews();

            Swal.fire({
                icon: 'success',
                title: 'เพิ่มวิชาเรียบร้อย',
                timer: 1200,
                showConfirmButton: false
            });
        }

        function handleDeleteSubject(id) {
            if (confirm('ยืนยันลบรายวิชานี้?')) {
                appState.subjects = appState.subjects.filter(s => s.id !== id);
                saveStateToStorage();
                renderAllViews();
            }
        }

        function renderStudentManagementTable() {
            const tbody = document.getElementById('table-students-list');
            const filterClass = document.getElementById('filter-student-class')?.value || 'ALL';
            const badge = document.getElementById('student-count-badge');

            let filtered = appState.students;
            if (filterClass !== 'ALL') {
                filtered = filtered.filter(s => s.classId === filterClass);
            }

            if (badge) badge.innerText = `แสดง ${filtered.length} รายการ`;

            if (filtered.length === 0) {
                tbody.innerHTML = `<tr><td colspan="5" class="text-center py-6 text-slate-400">ไม่พบข้อมูลนักเรียน</td></tr>`;
                return;
            }

            tbody.innerHTML = filtered.map(st => {
                const cls = appState.classes.find(c => c.id === st.classId);
                return `
                    <tr class="hover:bg-slate-50 transition">
                        <td class="p-3 font-bold text-slate-500">${st.number}</td>
                        <td class="p-3 font-mono text-xs font-semibold text-slate-700">${st.code}</td>
                        <td class="p-3 font-bold text-slate-800">${st.name}</td>
                        <td class="p-3"><span class="px-2.5 py-0.5 rounded-full text-xs font-semibold bg-indigo-50 text-indigo-700 border border-indigo-100">${cls ? cls.name : '-'}</span></td>
                        <td class="p-3 text-center">
                            <button onclick="handleDeleteStudent('${st.id}')" class="px-2.5 py-1 text-rose-600 hover:bg-rose-50 rounded-lg text-xs font-semibold transition">
                                <i class="fa-solid fa-trash mr-1"></i>ลบ
                            </button>
                        </td>
                    </tr>
                `;
            }).join('');
        }

        function importMockExcelStudents() {
            if (appState.classes.length === 0) {
                Swal.fire('ข้อผิดพลาด', 'กรุณาสร้างชั้นเรียนอย่างน้อย 1 ห้องก่อนนำเข้านักเรียน', 'warning');
                return;
            }

            const targetClassId = appState.classes[0].id;
            const targetClassName = appState.classes[0].name;

            const mockStudents = [
                { id: "st_m1", code: "30001", name: "เด็กชายอนันต์ สุขสวัสดิ์", number: 1, classId: targetClassId },
                { id: "st_m2", code: "30002", name: "เด็กหญิงนภา วงศ์สว่าง", number: 2, classId: targetClassId },
                { id: "st_m3", code: "30003", name: "นายธนกฤต เพชรแท้", number: 3, classId: targetClassId }
            ];

            appState.students.push(...mockStudents);
            saveStateToStorage();
            renderAllViews();

            Swal.fire({
                icon: 'success',
                title: 'นำเข้าข้อมูลจำลองสำเร็จ!',
                text: `เพิ่มนักเรียน 3 คนเข้าชั้นเรียน ${targetClassName} เรียบร้อยแล้ว`,
                timer: 1800,
                showConfirmButton: false
            });
        }

        // ==========================================
        // SESSION & QR CODE LOGIC
        // ==========================================

        function handleCreateSession(e) {
            e.preventDefault();
            const classId = document.getElementById('select-session-class').value;
            const subjectId = document.getElementById('select-session-subject').value;
            const sessionName = document.getElementById('input-session-name').value;
            const expireMinutes = parseInt(document.getElementById('select-session-expire').value);

            if (!classId || !subjectId) {
                Swal.fire('ข้อผิดพลาด', 'กรุณาเลือกชั้นเรียนและวิชาให้ครบถ้วน', 'warning');
                return;
            }

            const now = Date.now();
            const newSession = {
                id: "sess_" + now,
                classId,
                subjectId,
                sessionName,
                expireMinutes,
                startTime: now,
                endTime: now + (expireMinutes * 60 * 1000),
                qrToken: "QR_" + Math.random().toString(36).substr(2, 8).toUpperCase()
            };

            appState.activeSession = newSession;

            // Initialize attendance records for students in this class
            const classStudents = appState.students.filter(s => s.classId === classId);
            classStudents.forEach(st => {
                const exists = appState.attendance.find(a => a.sessionId === newSession.id && a.studentId === st.id);
                if (!exists) {
                    appState.attendance.push({
                        studentId: st.id,
                        sessionId: newSession.id,
                        status: 'RED',
                        otp: null,
                        scanTime: null,
                        verifyTime: null
                    });
                }
            });

            saveStateToStorage();
            renderAllViews();

            Swal.fire({
                icon: 'success',
                title: 'เปิดคาบเรียนเรียบร้อย',
                text: 'ระบบสร้าง QR Code พร้อมกำหนดเวลานับถอยหลังแล้ว',
                timer: 1800,
                showConfirmButton: false
            });
        }

        function refreshActiveQrcode() {
            if (!appState.activeSession) return;
            const now = Date.now();
            appState.activeSession.qrToken = "QR_" + Math.random().toString(36).substr(2, 8).toUpperCase();
            appState.activeSession.startTime = now;
            appState.activeSession.endTime = now + (appState.activeSession.expireMinutes * 60 * 1000);

            saveStateToStorage();
            renderActiveSessionQR();

            Swal.fire({
                icon: 'info',
                title: 'รีเฟรช QR Code แล้ว',
                timer: 1200,
                showConfirmButton: false
            });
        }

        function renderActiveSessionQR() {
            const activeContainer = document.getElementById('qr-active-container');
            const emptyContainer = document.getElementById('qr-empty-container');
            const qrElement = document.getElementById('qrcode-element');

            if (!appState.activeSession) {
                activeContainer.classList.add('hidden');
                emptyContainer.classList.remove('hidden');
                emptyContainer.classList.add('flex');
                return;
            }

            activeContainer.classList.remove('hidden');
            emptyContainer.classList.add('hidden');
            emptyContainer.classList.remove('flex');

            const cls = appState.classes.find(c => c.id === appState.activeSession.classId);
            const subj = appState.subjects.find(s => s.id === appState.activeSession.subjectId);

            document.getElementById('qr-session-title').innerText = `${cls ? cls.name : 'ชั้นเรียน'} - ${subj ? subj.name : 'วิชา'}`;
            document.getElementById('qr-session-period').innerText = `หัวข้อ: ${appState.activeSession.sessionName}`;

            // Render QRCode
            qrElement.innerHTML = '';
            const qrData = JSON.stringify({
                sessionId: appState.activeSession.id,
                token: appState.activeSession.qrToken
            });

            new QRCode(qrElement, {
                text: qrData,
                width: 180,
                height: 180,
                colorDark : "#0f172a",
                colorLight : "#ffffff",
                correctLevel : QRCode.CorrectLevel.H
            });
        }

        function startAutoRefreshTimer() {
            if (countdownTimer) clearInterval(countdownTimer);
            
            countdownTimer = setInterval(() => {
                if (!appState.activeSession) return;

                const timerDisplay = document.getElementById('qr-timer-display');
                if (!timerDisplay) return;

                const now = Date.now();
                const remaining = appState.activeSession.endTime - now;

                if (remaining <= 0) {
                    timerDisplay.innerText = "หมดอายุแล้ว";
                    timerDisplay.className = "text-3xl font-black text-rose-600 tracking-wider animate-pulse";
                } else {
                    const mins = Math.floor(remaining / 60000);
                    const secs = Math.floor((remaining % 60000) / 1000);
                    timerDisplay.innerText = `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
                    timerDisplay.className = "text-3xl font-black text-amber-600 tracking-wider";
                }
            }, 1000);
        }

        // ==========================================
        // DASHBOARD & OTP VERIFICATION LOGIC
        // ==========================================

        function renderDashboardStats() {
            if (!appState.activeSession) {
                document.getElementById('stat-total-students').innerText = '0';
                document.getElementById('stat-present-today').innerText = '0';
                document.getElementById('stat-pending-today').innerText = '0';
                document.getElementById('stat-absent-today').innerText = '0';
                document.getElementById('dash-active-session-title').innerText = 'ไม่มีคาบเรียนที่เปิดอยู่ในขณะนี้';
                return;
            }

            const currentClassStudents = appState.students.filter(s => s.classId === appState.activeSession.classId);
            const total = currentClassStudents.length;

            let present = 0, pending = 0, absent = 0;

            currentClassStudents.forEach(st => {
                const rec = appState.attendance.find(a => a.studentId === st.id && a.sessionId === appState.activeSession.id);
                if (!rec || rec.status === 'RED') {
                    absent++;
                } else if (rec.status === 'YELLOW') {
                    pending++;
                } else if (rec.status === 'GREEN') {
                    present++;
                }
            });

            const cls = appState.classes.find(c => c.id === appState.activeSession.classId);
            const subj = appState.subjects.find(s => s.id === appState.activeSession.subjectId);

            document.getElementById('stat-total-students').innerText = total;
            document.getElementById('stat-present-today').innerText = present;
            document.getElementById('stat-pending-today').innerText = pending;
            document.getElementById('stat-absent-today').innerText = absent;
            document.getElementById('dash-active-session-title').innerText = `${cls ? cls.name : ''} | ${subj ? subj.name : ''} (${appState.activeSession.sessionName})`;
        }

        function renderDashboardAttendanceTable() {
            const tbody = document.getElementById('table-dashboard-attendance');
            tbody.innerHTML = '';

            if (!appState.activeSession) {
                tbody.innerHTML = `<tr><td colspan="7" class="text-center py-8 text-slate-400">กรุณาเปิดคาบเรียนเพื่อแสดงข้อมูล</td></tr>`;
                return;
            }

            const classStudents = appState.students
                .filter(s => s.classId === appState.activeSession.classId)
                .sort((a, b) => a.number - b.number);

            if (classStudents.length === 0) {
                tbody.innerHTML = `<tr><td colspan="7" class="text-center py-8 text-slate-400">ไม่มีรายชื่อนักเรียนในชั้นเรียนนี้</td></tr>`;
                return;
            }

            classStudents.forEach(st => {
                const att = appState.attendance.find(a => a.studentId === st.id && a.sessionId === appState.activeSession.id);
                const status = att ? att.status : 'RED';
                
                let badge = '';
                if (status === 'GREEN') {
                    badge = `<span class="inline-flex items-center px-2.5 py-1 rounded-full text-xs font-bold bg-emerald-100 text-emerald-800"><i class="fa-solid fa-circle text-[8px] mr-1.5 text-emerald-500"></i>🟢 เข้าเรียนแล้ว</span>`;
                } else if (status === 'YELLOW') {
                    badge = `<span class="inline-flex items-center px-2.5 py-1 rounded-full text-xs font-bold bg-amber-100 text-amber-800"><i class="fa-solid fa-circle text-[8px] mr-1.5 text-amber-500 animate-pulse"></i>🟡 ได้รับ OTP แล้ว</span>`;
                } else {
                    badge = `<span class="inline-flex items-center px-2.5 py-1 rounded-full text-xs font-bold bg-rose-100 text-rose-800"><i class="fa-solid fa-circle text-[8px] mr-1.5 text-rose-500"></i>🔴 ยังไม่เช็คชื่อ</span>`;
                }

                let timeText = '-';
                if (att && att.verifyTime) timeText = `ยืนยัน: ${att.verifyTime}`;
                else if (att && att.scanTime) timeText = `สแกน: ${att.scanTime}`;

                let actionBtn = '-';
                if (status === 'YELLOW') {
                    actionBtn = `<button onclick="confirmStudentAttendance('${st.id}')" class="px-3 py-1 bg-emerald-600 hover:bg-emerald-700 text-white rounded-xl text-xs font-bold shadow-sm transition">อนุมัติ OTP</button>`;
                } else if (status === 'RED') {
                    actionBtn = `<button onclick="forceCheckinStudent('${st.id}')" class="px-2.5 py-1 bg-slate-100 hover:bg-slate-200 text-slate-600 rounded-xl text-xs font-semibold transition">เช็คชื่อให้</button>`;
                }

                const tr = document.createElement('tr');
                tr.className = "hover:bg-slate-50 transition";
                tr.innerHTML = `
                    <td class="p-3 font-bold text-slate-500">${st.number}</td>
                    <td class="p-3 font-mono text-xs font-semibold">${st.code}</td>
                    <td class="p-3 font-bold text-slate-800">${st.name}</td>
                    <td class="p-3 text-center font-mono font-black text-indigo-600 text-base">${att && att.otp ? att.otp : '-'}</td>
                    <td class="p-3 text-center text-xs text-slate-500 font-semibold">${timeText}</td>
                    <td class="p-3 text-center">${badge}</td>
                    <td class="p-3 text-center">${actionBtn}</td>
                `;
                tbody.appendChild(tr);
            });
        }

        function renderPendingOTPList() {
            const container = document.getElementById('pending-otp-list');
            const badge = document.getElementById('pending-count-badge');
            container.innerHTML = '';

            if (!appState.activeSession) {
                badge.innerText = '0 รายการ';
                container.innerHTML = '<p class="text-xs text-slate-400 text-center py-4">ไม่มีคาบเรียนที่เปิดอยู่</p>';
                return;
            }

            const pendingList = appState.attendance.filter(a => a.sessionId === appState.activeSession.id && a.status === 'YELLOW');
            badge.innerText = `${pendingList.length} รายการ`;

            if (pendingList.length === 0) {
                container.innerHTML = '<p class="text-xs text-slate-400 text-center py-4">ไม่มีรายการรอยืนยัน OTP ในขณะนี้</p>';
                return;
            }

            pendingList.forEach(item => {
                const student = appState.students.find(s => s.id === item.studentId);
                if (!student) return;

                const div = document.createElement('div');
                div.className = "flex items-center justify-between p-3.5 bg-amber-50/80 border border-amber-200 rounded-2xl shadow-sm";
                div.innerHTML = `
                    <div class="flex items-center space-x-3">
                        <span class="text-xl font-mono font-black text-amber-700 bg-amber-100/80 border border-amber-300 px-3 py-1 rounded-xl shadow-inner">${item.otp}</span>
                        <div>
                            <p class="text-xs font-bold text-slate-800">${student.name} (เลขที่ ${student.number})</p>
                            <p class="text-[10px] text-slate-500">เวลาสแกน: ${item.scanTime}</p>
                        </div>
                    </div>
                    <button onclick="confirmStudentAttendance('${student.id}')" class="px-3.5 py-2 bg-emerald-600 hover:bg-emerald-700 text-white rounded-xl text-xs font-bold shadow-md shadow-emerald-500/20 transition flex items-center gap-1">
                        <i class="fa-solid fa-check"></i>อนุมัติ
                    </button>
                `;
                container.appendChild(div);
            });
        }

        function handleManualVerifyOTP(e) {
            e.preventDefault();
            const input = document.getElementById('input-verify-otp');
            const code = input.value.trim();

            if (!code || code.length !== 6) {
                Swal.fire('ข้อผิดพลาด', 'กรุณากรอกรหัส OTP 6 หลักให้ถูกต้อง', 'error');
                return;
            }

            if (!appState.activeSession) {
                Swal.fire('ข้อผิดพลาด', 'ไม่มีคาบเรียนที่กำลังดำเนินอยู่', 'error');
                return;
            }

            const record = appState.attendance.find(a => a.sessionId === appState.activeSession.id && a.otp === code);

            if (!record) {
                Swal.fire('ไม่พบรหัส OTP', 'รหัส OTP ไม่ถูกต้อง หรือนักเรียนยังไม่ได้สแกน QR Code', 'error');
                return;
            }

            if (record.status === 'GREEN') {
                Swal.fire('แจ้งเตือน', 'รหัส OTP นี้ได้รับการยืนยันเข้าเรียนไปเรียบร้อยแล้ว', 'warning');
                return;
            }

            record.status = 'GREEN';
            record.verifyTime = new Date().toLocaleTimeString('th-TH');

            saveStateToStorage();
            renderAllViews();
            input.value = '';

            confetti({ particleCount: 60, spread: 60, origin: { y: 0.7 } });

            const student = appState.students.find(s => s.id === record.studentId);
            Swal.fire({
                icon: 'success',
                title: 'ยืนยันเข้าเรียนสำเร็จ!',
                text: `${student ? student.name : 'นักเรียน'} เช็คชื่อเรียบร้อยแล้ว 🟢`,
                timer: 1800,
                showConfirmButton: false
            });
        }

        function confirmStudentAttendance(studentId) {
            const record = appState.attendance.find(a => a.sessionId === appState.activeSession.id && a.studentId === studentId);
            if (record) {
                record.status = 'GREEN';
                record.verifyTime = new Date().toLocaleTimeString('th-TH');
                saveStateToStorage();
                renderAllViews();

                confetti({ particleCount: 50, spread: 50, origin: { y: 0.7 } });

                const student = appState.students.find(s => s.id === studentId);
                Swal.fire({
                    icon: 'success',
                    title: 'อนุมัติการเข้าเรียนแล้ว',
                    text: `${student ? student.name : 'นักเรียน'} เปลี่ยนสถานะเป็น 🟢 เข้าเรียนแล้ว`,
                    timer: 1500,
                    showConfirmButton: false
                });
            }
        }

        function forceCheckinStudent(studentId) {
            if (!appState.activeSession) return;
            let record = appState.attendance.find(a => a.sessionId === appState.activeSession.id && a.studentId === studentId);
            
            if (!record) {
                record = {
                    studentId: studentId,
                    sessionId: appState.activeSession.id,
                    status: 'GREEN',
                    otp: Math.floor(100000 + Math.random() * 900000).toString(),
                    scanTime: new Date().toLocaleTimeString('th-TH'),
                    verifyTime: new Date().toLocaleTimeString('th-TH')
                };
                appState.attendance.push(record);
            } else {
                record.status = 'GREEN';
                record.verifyTime = new Date().toLocaleTimeString('th-TH');
                if (!record.otp) record.otp = Math.floor(100000 + Math.random() * 900000).toString();
            }

            saveStateToStorage();
            renderAllViews();
        }

        // ==========================================
        // STUDENT VIEW & MOBILE SIMULATION LOGIC
        // ==========================================

        function renderStudentIdentitySelect() {
            const select = document.getElementById('select-student-identity');
            if (!select) return;
            select.innerHTML = appState.students.map(s => {
                const cls = appState.classes.find(c => c.id === s.classId);
                return `<option value="${s.id}" ${s.id === appState.selectedStudentId ? 'selected' : ''}>${s.name} (${cls ? cls.name : ''})</option>`;
            }).join('');
        }

        function handleStudentIdentityChange() {
            appState.selectedStudentId = document.getElementById('select-student-identity').value;
            saveStateToStorage();
            renderStudentView();
        }

        function renderStudentView() {
            const container = document.getElementById('student-content-container');
            const student = appState.students.find(s => s.id === appState.selectedStudentId);

            if (!student) {
                container.innerHTML = `<div class="text-center py-6 text-slate-400">ไม่พบข้อมูลนักเรียน กรุณาเพิ่มนักเรียนในระบบครู</div>`;
                return;
            }

            const studentClass = appState.classes.find(c => c.id === student.classId);
            const activeSession = appState.activeSession;

            const isSessionForStudent = activeSession && activeSession.classId === student.classId;
            const activeSubject = isSessionForStudent ? appState.subjects.find(s => s.id === activeSession.subjectId) : null;
            
            let attendanceRecord = isSessionForStudent 
                ? appState.attendance.find(a => a.sessionId === activeSession.id && a.studentId === student.id)
                : null;

            let contentHTML = `
                <!-- Student Card Profile -->
                <div class="bg-indigo-50/80 border border-indigo-100 rounded-2xl p-4 flex items-center justify-between shadow-sm">
                    <div>
                        <p class="text-[10px] font-bold text-indigo-500 uppercase tracking-wider">นักเรียน</p>
                        <h3 class="font-bold text-slate-800 text-base">${student.name}</h3>
                        <p class="text-xs text-slate-500">รหัส: ${student.code} | เลขที่: ${student.number} | ชั้น: ${studentClass ? studentClass.name : '-'}</p>
                    </div>
                    <div class="w-10 h-10 rounded-2xl bg-indigo-600 text-white flex items-center justify-center font-black text-sm shadow-md">
                        ${student.number}
                    </div>
                </div>
            `;

            if (!isSessionForStudent) {
                contentHTML += `
                    <div class="text-center py-8 space-y-3 bg-slate-50 rounded-2xl border border-dashed border-slate-200">
                        <i class="fa-solid fa-moon text-3xl text-slate-300"></i>
                        <p class="text-sm font-bold text-slate-600">ยังไม่มีการเปิดคาบเรียนในชั้นของคุณ</p>
                        <p class="text-xs text-slate-400 max-w-xs mx-auto">เมื่อคุณครูเปิดคาบเรียนใหม่ คุณจะสามารถกดสแกน QR เพื่อรับรหัส OTP เข้าเรียนได้ทันที</p>
                    </div>
                `;
            } else {
                const now = Date.now();
                const isExpired = activeSession.endTime < now;
                const status = attendanceRecord ? attendanceRecord.status : 'RED';

                contentHTML += `
                    <!-- Active Subject Info -->
                    <div class="border-b border-slate-100 pb-3 space-y-1">
                        <div class="flex items-center justify-between">
                            <span class="text-xs font-bold text-indigo-600">วิชาที่กำลังเช็คชื่อ</span>
                            <span class="text-[10px] bg-indigo-100 text-indigo-700 font-bold px-2.5 py-0.5 rounded-full">LIVE</span>
                        </div>
                        <h4 class="font-extrabold text-slate-800 text-lg">${activeSubject ? activeSubject.name : ''}</h4>
                        <p class="text-xs text-slate-500">${activeSession.sessionName}</p>
                    </div>
                `;

                if (status === 'RED') {
                    contentHTML += `
                        <div class="text-center space-y-4 py-4">
                            <div class="p-4 bg-rose-50 rounded-2xl border border-rose-100 space-y-2">
                                <i class="fa-solid fa-circle-exclamation text-rose-500 text-2xl"></i>
                                <h5 class="font-bold text-rose-800 text-sm">สถานะ: 🔴 ยังไม่เช็คชื่อ</h5>
                                <p class="text-xs text-rose-600">กดปุ่มสแกนด้านล่างเพื่อรับรหัส OTP สำหรับแจ้งคุณครู</p>
                            </div>

                            <button onclick="studentSimulateScanQR()" ${isExpired ? 'disabled' : ''} class="w-full py-3.5 bg-indigo-600 hover:bg-indigo-700 text-white font-bold rounded-2xl text-sm transition shadow-lg shadow-indigo-500/25 flex items-center justify-center space-x-2 ${isExpired ? 'opacity-50 cursor-not-allowed' : ''}">
                                <i class="fa-solid fa-qrcode text-base"></i>
                                <span>${isExpired ? 'QR Code หมดอายุแล้ว' : 'จำลองสแกน QR Code เพื่อรับ OTP'}</span>
                            </button>
                        </div>
                    `;
                } else if (status === 'YELLOW') {
                    contentHTML += `
                        <div class="text-center space-y-4 py-2">
                            <div class="p-4 bg-amber-50/90 rounded-2xl border border-amber-200 space-y-3">
                                <span class="inline-flex items-center px-3 py-1 rounded-full text-xs font-bold bg-amber-100 text-amber-800">
                                    <span class="w-2 h-2 mr-1.5 bg-amber-500 rounded-full animate-ping"></span>
                                    สถานะ: 🟡 ได้รับ OTP แล้ว (แจ้งครู)
                                </span>

                                <div class="bg-white p-4 rounded-2xl border-2 border-amber-300 shadow-inner">
                                    <p class="text-[11px] text-slate-400 font-bold mb-1">รหัสยืนยันเข้าเรียน (OTP 6 หลัก)</p>
                                    <p class="text-4xl font-mono font-black text-amber-600 tracking-widest my-1">${attendanceRecord.otp}</p>
                                    <p class="text-[10px] text-slate-400">สแกนเมื่อ: ${attendanceRecord.scanTime}</p>
                                </div>

                                <button onclick="copyOTPToClipboard('${attendanceRecord.otp}')" class="w-full py-2.5 bg-amber-600 hover:bg-amber-700 text-white text-xs font-bold rounded-xl transition flex items-center justify-center gap-2 shadow-sm">
                                    <i class="fa-solid fa-copy"></i>
                                    <span>คัดลอกรหัส OTP</span>
                                </button>
                            </div>
                        </div>
                    `;
                } else if (status === 'GREEN') {
                    contentHTML += `
                        <div class="text-center space-y-3 py-4">
                            <div class="p-5 bg-emerald-50 rounded-2xl border border-emerald-200 space-y-2">
                                <div class="w-12 h-12 bg-emerald-500 text-white rounded-full flex items-center justify-center mx-auto text-xl shadow-md">
                                    <i class="fa-solid fa-check"></i>
                                </div>
                                <h5 class="font-extrabold text-emerald-800 text-lg">สถานะ: 🟢 เข้าเรียนแล้ว!</h5>
                                <p class="text-xs text-emerald-600">คุณครูได้ยืนยันการเข้าเรียนของคุณเรียบร้อยแล้ว</p>
                                <p class="text-[11px] font-mono text-slate-400 pt-1">เวลายืนยัน: ${attendanceRecord.verifyTime || '-'}</p>
                            </div>
                        </div>
                    `;
                }
            }

            container.innerHTML = contentHTML;
        }

        function studentSimulateScanQR() {
            if (!appState.activeSession) return;
            const studentId = appState.selectedStudentId;

            if (appState.activeSession.endTime < Date.now()) {
                Swal.fire('หมดเวลา', 'QR Code สำหรับคาบนี้หมดอายุแล้ว ไม่สามารถเช็คชื่อได้', 'error');
                return;
            }

            let record = appState.attendance.find(a => a.sessionId === appState.activeSession.id && a.studentId === studentId);

            const generatedOTP = Math.floor(100000 + Math.random() * 900000).toString();

            if (!record) {
                record = {
                    studentId,
                    sessionId: appState.activeSession.id,
                    status: 'YELLOW',
                    otp: generatedOTP,
                    scanTime: new Date().toLocaleTimeString('th-TH'),
                    verifyTime: null
                };
                appState.attendance.push(record);
            } else {
                record.status = 'YELLOW';
                record.otp = generatedOTP;
                record.scanTime = new Date().toLocaleTimeString('th-TH');
            }

            saveStateToStorage();
            renderAllViews();

            Swal.fire({
                icon: 'success',
                title: 'สแกนสำเร็จ!',
                text: `คุณได้รับรหัส OTP: ${generatedOTP}`,
                timer: 1800,
                showConfirmButton: false
            });
        }

        function simulateStudentScan() {
            switchRole('student');
            studentSimulateScanQR();
        }

        function copyOTPToClipboard(otp) {
            const tempInput = document.createElement('input');
            tempInput.value = otp;
            document.body.appendChild(tempInput);
            tempInput.select();
            document.execCommand('copy');
            document.body.removeChild(tempInput);

            Swal.fire({
                icon: 'success',
                title: 'คัดลอกรหัสแล้ว',
                text: `รหัส OTP ${otp} ถูกคัดลอกเรียบร้อย`,
                timer: 1200,
                showConfirmButton: false
            });
        }

        // ==========================================
        // REPORTS & STATISTICS TAB
        // ==========================================

        function renderReportTable() {
            const classId = document.getElementById('report-filter-class')?.value;
            const tbody = document.getElementById('table-report-attendance');
            const summaryContainer = document.getElementById('report-summary-cards');

            if (!tbody) return;

            if (!classId) {
                tbody.innerHTML = `<tr><td colspan="7" class="text-center py-6 text-slate-400">กรุณาเลือกชั้นเรียนเพื่อดูรายงาน</td></tr>`;
                return;
            }

            const studentsInClass = appState.students.filter(s => s.classId === classId).sort((a,b) => a.number - b.number);
            const targetClass = appState.classes.find(c => c.id === classId);

            let greenCount = 0, yellowCount = 0, redCount = 0;

            studentsInClass.forEach(st => {
                const att = appState.attendance.find(a => a.studentId === st.id && (appState.activeSession ? a.sessionId === appState.activeSession.id : true));
                if (att?.status === 'GREEN') greenCount++;
                else if (att?.status === 'YELLOW') yellowCount++;
                else redCount++;
            });

            if (summaryContainer) {
                summaryContainer.innerHTML = `
                    <div class="bg-emerald-50 border border-emerald-200 p-4 rounded-2xl text-center">
                        <p class="text-xs font-bold text-emerald-600">เข้าเรียนแล้ว (🟢)</p>
                        <p class="text-2xl font-black text-emerald-700">${greenCount} คน</p>
                    </div>
                    <div class="bg-amber-50 border border-amber-200 p-4 rounded-2xl text-center">
                        <p class="text-xs font-bold text-amber-600">รอยืนยัน OTP (🟡)</p>
                        <p class="text-2xl font-black text-amber-700">${yellowCount} คน</p>
                    </div>
                    <div class="bg-rose-50 border border-rose-200 p-4 rounded-2xl text-center">
                        <p class="text-xs font-bold text-rose-600">ยังไม่เช็คชื่อ (🔴)</p>
                        <p class="text-2xl font-black text-rose-700">${redCount} คน</p>
                    </div>
                `;
            }

            if (studentsInClass.length === 0) {
                tbody.innerHTML = `<tr><td colspan="7" class="text-center py-6 text-slate-400">ไม่มีรายชื่อนักเรียนในชั้นเรียนนี้</td></tr>`;
                return;
            }

            tbody.innerHTML = studentsInClass.map(st => {
                const att = appState.attendance.find(a => a.studentId === st.id && (appState.activeSession ? a.sessionId === appState.activeSession.id : true));
                const status = att ? att.status : 'RED';

                let badge = '<span class="text-rose-600 font-bold">🔴 ยังไม่เช็คชื่อ</span>';
                if (status === 'GREEN') badge = '<span class="text-emerald-600 font-bold">🟢 เข้าเรียนแล้ว</span>';
                else if (status === 'YELLOW') badge = '<span class="text-amber-600 font-bold">🟡 ได้รหัส OTP แล้ว</span>';

                return `
                    <tr class="hover:bg-slate-50 transition">
                        <td class="p-3 font-bold text-slate-500">${st.number}</td>
                        <td class="p-3 font-mono text-xs">${st.code}</td>
                        <td class="p-3 font-bold text-slate-800">${st.name}</td>
                        <td class="p-3 font-semibold text-slate-600">${targetClass ? targetClass.name : '-'}</td>
                        <td class="p-3 text-center">${badge}</td>
                        <td class="p-3 text-center text-xs text-slate-500">${att?.scanTime || '-'}</td>
                        <td class="p-3 text-center text-xs text-slate-500">${att?.verifyTime || '-'}</td>
                    </tr>
                `;
            }).join('');
        }

        function exportToCSV() {
            if (!appState.activeSession) {
                Swal.fire('ข้อผิดพลาด', 'ไม่มีข้อมูลคาบเรียนเพื่อส่งออก', 'error');
                return;
            }

            const cls = appState.classes.find(c => c.id === appState.activeSession.classId);
            const subj = appState.subjects.find(s => s.id === appState.activeSession.subjectId);

            let csvContent = "\uFEFF"; // UTF-8 BOM for Thai Language support
            csvContent += `รายงานการเช็คชื่อเข้าเรียน\n`;
            csvContent += `ชั้นเรียน,${cls ? cls.name : ''}\n`;
            csvContent += `วิชา,${subj ? subj.name : ''}\n`;
            csvContent += `คาบเรียน,${appState.activeSession.sessionName}\n`;
            csvContent += `วันที่ส่งออก,${new Date().toLocaleDateString('th-TH')}\n\n`;

            csvContent += `เลขที่,รหัสนักเรียน,ชื่อ-นามสกุล,รหัส OTP,เวลาสแกน,เวลายืนยัน,สถานะ\n`;

            const classStudents = appState.students
                .filter(s => s.classId === appState.activeSession.classId)
                .sort((a, b) => a.number - b.number);

            classStudents.forEach(st => {
                const att = appState.attendance.find(a => a.studentId === st.id && a.sessionId === appState.activeSession.id);
                let statusText = 'ยังไม่เช็คชื่อ';
                if (att?.status === 'GREEN') statusText = 'เข้าเรียนแล้ว';
                else if (att?.status === 'YELLOW') statusText = 'สแกนแล้ว (รออนุมัติ)';

                csvContent += `"${st.number}","${st.code}","${st.name}","${att?.otp || '-'}","${att?.scanTime || '-'}","${att?.verifyTime || '-'}","${statusText}"\n`;
            });

            const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
            const url = URL.createObjectURL(blob);
            const link = document.createElement("a");
            link.setAttribute("href", url);
            link.setAttribute("download", `รายงานเช็คชื่อ_${cls ? cls.name : 'ชั้นเรียน'}_${new Date().toISOString().slice(0,10)}.csv`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        }

        // Student Modal Handlers
        function openAddStudentModal() {
            document.getElementById('modal-add-student').classList.remove('hidden');
        }

        function closeAddStudentModal() {
            document.getElementById('modal-add-student').classList.add('hidden');
        }

        function handleSaveStudent(e) {
            e.preventDefault();
            const code = document.getElementById('input-student-code').value.trim();
            const name = document.getElementById('input-student-name').value.trim();
            const number = parseInt(document.getElementById('input-student-number').value);
            const classId = document.getElementById('select-student-class').value;

            appState.students.push({
                id: "st_" + Date.now(),
                code,
                name,
                number,
                classId
            });

            saveStateToStorage();
            closeAddStudentModal();
            renderAllViews();

            Swal.fire({
                icon: 'success',
                title: 'บันทึกเรียบร้อย',
                text: 'เพิ่มนักเรียนใหม่เข้าระบบสำเร็จ',
                timer: 1500,
                showConfirmButton: false
            });
        }

        function handleDeleteStudent(id) {
            if (confirm('ยืนยันการลบนักเรียนคนนี้?')) {
                appState.students = appState.students.filter(s => s.id !== id);
                saveStateToStorage();
                renderAllViews();
            }
        }

        // Role Switcher Navigation
        function switchRole(role) {
            appState.currentRole = role;
            const teacherView = document.getElementById('teacher-view');
            const studentView = document.getElementById('student-view');
            const btnTeacher = document.getElementById('btn-role-teacher');
            const btnStudent = document.getElementById('btn-role-student');

            if (role === 'teacher') {
                teacherView.classList.remove('hidden');
                studentView.classList.add('hidden');
                
                btnTeacher.className = "px-3.5 py-1.5 rounded-xl text-xs sm:text-sm font-semibold transition-all duration-200 flex items-center space-x-2 bg-indigo-600 text-white shadow-sm";
                btnStudent.className = "px-3.5 py-1.5 rounded-xl text-xs sm:text-sm font-semibold transition-all duration-200 flex items-center space-x-2 text-slate-400 hover:text-white";
            } else {
                teacherView.classList.add('hidden');
                studentView.classList.remove('hidden');

                btnStudent.className = "px-3.5 py-1.5 rounded-xl text-xs sm:text-sm font-semibold transition-all duration-200 flex items-center space-x-2 bg-indigo-600 text-white shadow-sm";
                btnTeacher.className = "px-3.5 py-1.5 rounded-xl text-xs sm:text-sm font-semibold transition-all duration-200 flex items-center space-x-2 text-slate-400 hover:text-white";
                
                renderStudentView();
            }
        }

        // Teacher Tab Navigation Switcher
        function switchTeacherTab(tabId) {
            document.querySelectorAll('.teacher-tab-content').forEach(el => el.classList.add('hidden'));
            document.querySelectorAll('.teacher-tab-btn').forEach(btn => {
                btn.className = "teacher-tab-btn flex-1 min-w-[110px] sm:min-w-[130px] px-3.5 py-2.5 rounded-xl flex items-center justify-center space-x-2 text-slate-600 hover:bg-slate-100 transition";
            });

            document.getElementById(tabId).classList.remove('hidden');
            const activeBtn = document.getElementById('nav-' + tabId);
            if (activeBtn) {
                activeBtn.className = "teacher-tab-btn flex-1 min-w-[110px] sm:min-w-[130px] px-3.5 py-2.5 rounded-xl flex items-center justify-center space-x-2 bg-indigo-50 text-indigo-700 font-semibold transition";
            }
        }
    </script>
</body>
</html>
