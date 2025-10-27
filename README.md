<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>جمعية المجد للماء - استعلام الفواتير</title>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-firestore-compat.js"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { 
            font-family: Arial, sans-serif; 
            background: #e0f7fa; 
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }
        .container { 
            width: 100%; 
            max-width: 400px; 
            background: white; 
            border-radius: 15px; 
            box-shadow: 0 5px 15px rgba(0,0,0,0.1);
            overflow: hidden;
        }
        .header { 
            background: #00796b; 
            color: white; 
            padding: 20px; 
            text-align: center; 
            position: relative;
        }
        .header h1 { font-size: 22px; margin-bottom: 5px; }
        .user-btn { 
            position: absolute; 
            top: 10px; 
            right: 10px; 
            background: rgba(255,255,255,0.2); 
            border: none; 
            color: white; 
            padding: 5px 10px; 
            border-radius: 10px; 
            font-size: 12px; 
            cursor: pointer;
        }
        .hidden-admin { 
            position: absolute; 
            top: 10px; 
            left: 10px; 
            width: 20px; 
            height: 20px; 
            background: transparent; 
            border: none; 
            cursor: pointer;
            opacity: 0.1;
        }
        .content { padding: 20px; }
        .section { display: none; }
        .active { display: block; }
        input, button, select { 
            width: 100%; 
            padding: 12px; 
            margin: 8px 0; 
            border: 1px solid #ddd; 
            border-radius: 8px; 
            font-size: 16px;
        }
        button { 
            background: #00796b; 
            color: white; 
            border: none; 
            cursor: pointer; 
            font-weight: bold;
        }
        .btn-secondary { background: #78909c; }
        .status { 
            padding: 10px; 
            margin: 10px 0; 
            border-radius: 5px; 
            text-align: center;
        }
        .success { background: #e8f5e9; color: #2e7d32; }
        .error { background: #ffebee; color: #c62828; }
        .welcome-btns { margin: 20px 0; }
        .welcome-btn { 
            display: block; 
            width: 100%; 
            padding: 15px; 
            margin: 10px 0; 
            background: #f5f5f5; 
            border: 1px solid #ddd; 
            border-radius: 8px; 
            cursor: pointer;
            font-size: 16px;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <button class="hidden-admin" onclick="showAdminLogin()"></button>
            <button class="user-btn" onclick="showUserAuth()">🔐 مستخدم مسجل</button>
            <h1>💧 جمعية المجد للماء</h1>
            <p>التطبيق الرسمي لاستعلام الفواتير</p>
        </div>

        <div class="content">
            <!-- الصفحة الرئيسية -->
            <div id="welcome" class="section active">
                <h3 style="text-align: center; margin-bottom: 20px; color: #00796b;">مرحباً بك</h3>
                <div class="welcome-btns">
                    <button class="welcome-btn" onclick="showConsumer()">👤 مستخدم زائر</button>
                    <button class="welcome-btn" onclick="showUserAuth()">🔐 مستخدم مسجل</button>
                </div>
            </div>

            <!-- المستخدم الزائر -->
            <div id="consumer" class="section">
                <button class="btn-secondary" onclick="showWelcome()">← العودة</button>
                <input type="number" id="meterNumber" placeholder="رقم العداد (1-160)" min="1" max="160">
                <button onclick="searchBill()">🔍 استعلام عن الفاتورة</button>
                <div id="searchResults"></div>
                <div id="searchStatus" class="status"></div>
            </div>

            <!-- تسجيل الدخول -->
            <div id="userAuth" class="section">
                <button class="btn-secondary" onclick="showWelcome()">← العودة</button>
                <h4 style="text-align: center; margin: 15px 0; color: #00796b;">تسجيل الدخول</h4>
                <input type="text" id="loginUser" placeholder="اسم المستخدم">
                <input type="password" id="loginPass" placeholder="كلمة المرور">
                <button onclick="userLogin()">🚪 دخول</button>
                <div id="userStatus" class="status"></div>
            </div>

            <!-- المشرف -->
            <div id="adminLogin" class="section">
                <h4 style="text-align: center; margin: 15px 0; color: #d32f2f;">دخول المشرف</h4>
                <input type="text" id="adminUser" placeholder="Adminalmajd">
                <input type="password" id="adminPass" placeholder="كلمة المرور">
                <button onclick="adminLogin()">🚪 دخول المشرف</button>
                <div id="adminStatus" class="status"></div>
            </div>

            <!-- لوحة المشرف -->
            <div id="adminPanel" class="section">
                <button class="btn-secondary" onclick="showWelcome()">← الرئيسية</button>
                <h4 style="text-align: center; margin: 15px 0; color: #d32f2f;">لوحة المشرف</h4>
                <input type="number" id="adminMeterId" placeholder="رقم العداد">
                <input type="number" id="adminConsumption" placeholder="الاستهلاك (طن)" step="0.1">
                <button onclick="saveMeterData()">💾 حفظ البيانات</button>
                <div id="adminResults"></div>
            </div>
        </div>
    </div>

    <script>
        // إعدادات Firebase
        const firebaseConfig = {
            apiKey: "AIzaSyB_MmcEbHoq9K4DG1GKhcRr6axgPCg0CGg",
            authDomain: "water-association-app-441bb.firebaseapp.com",
            projectId: "water-association-app-441bb",
            storageBucket: "water-association-app-441bb.firebasestorage.app",
            messagingSenderId: "181162681371",
            appId: "1:181162681371:web:7c2b93cb5f6b7b0399c4be"
        };

        // التهيئة
        const app = firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();

        // دوال التنقل
        function showSection(section) {
            document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
            document.getElementById(section).classList.add('active');
        }
        function showWelcome() { showSection('welcome'); }
        function showConsumer() { showSection('consumer'); }
        function showUserAuth() { showSection('userAuth'); }
        function showAdminLogin() { showSection('adminLogin'); }

        // دوال المستخدم
        async function userLogin() {
            const username = document.getElementById('loginUser').value;
            const password = document.getElementById('loginPass').value;
            const status = document.getElementById('userStatus');

            // التحقق من المشرف
            if (username === "Adminalmajd" && password === "0700380091") {
                showSection('adminLogin');
                document.getElementById('adminUser').value = username;
                document.getElementById('adminPass').value = password;
                return;
            }

            showStatus(status, 'جاري التحقق...', '');
            
            try {
                const userDoc = await db.collection('users').doc(username).get();
                if (userDoc.exists && userDoc.data().password === password) {
                    showStatus(status, 'تم الدخول بنجاح', 'success');
                    showSection('consumer');
                } else {
                    showStatus(status, 'بيانات الدخول غير صحيحة', 'error');
                }
            } catch (error) {
                showStatus(status, 'خطأ في الاتصال', 'error');
            }
        }

        // دوال
