<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>إدارة المستخدمين وقائمة المسجلين</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">
    <style>
        body {
            display: flex;
            flex-direction: column;
            min-height: 100vh;
            background: url('https://i.imgur.com/2NdLVq3.png') no-repeat center center fixed;
            background-size: cover;
        }
        .container {
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 30px;
            background-color: rgba(0, 0, 0, 0.7);
            border-radius: 15px;
            margin-top: 50px;
        }
        .box {
            width: 150px;
            height: 80px;
            margin: 10px;
            border-radius: 10px; 
            background-color: #6a0dad;
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            text-align: center;
            font-weight: bold;
            font-size: 18px;
            position: relative; 
            overflow: hidden; 
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
            transition: all 0.3s; 
        }
        .box:hover {
            transform: translateY(-5px) scale(1.05);
            background-color: #550a8a;
        }
        .title {
            text-align: center;
            margin: 20px 0;
            font-size: 28px;
            font-weight: bold;
            color: white;
            text-shadow: 1px 1px 3px rgba(0, 0, 0, 0.7);
        }
        footer {
            text-align: center;
            padding: 20px 0;
            background-color: rgba(0, 0, 0, 0.7);
            color: #fff;
            margin-top: auto;
            font-weight: bold;
        }
        .flex-container {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
        }
        .waiting-line {
            width: 100%;
            height: 4px;
            background-color: yellow;
            margin: 20px 0;
            text-align: center;
            font-weight: bold;
            color: black;
            display: none;
        }
        .result-count {
            margin: 20px 0;
            color: white;
            font-weight: bold;
            padding: 20px; 
            border: 2px solid #6a0dad;
            border-radius: 10px; 
            background: rgba(0, 0, 0, 0.5); 
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
        }
        .loading {
            display: none;
            color: white;
            font-size: 18px;
            margin: 20px 0;
            animation: fadeIn 0.5s;
        }
        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }
        .remove-btn {
            color: red; /* لون زر الإكس */
            cursor: pointer;
            font-size: 24px; /* حجم زر الإكس */
            margin-left: 10px;
        }
    </style>
</head>
<body>
    <h2>إدارة المستخدمين</h2>
    <div class="user-list" id="userList"></div>

    <div class="container">
        <div class="title">أسماء المشتركين</div>
        <input type="text" id="searchInput" class="form-control mb-3" placeholder="ابحث عن اسم..." style="max-width: 400px;">
        <div id="loadingIndicator" class="loading">جاري تحميل البيانات...</div>
        <div id="resultCount" class="result-count"></div>
        <div id="approvedRegistrantsList" class="flex-container"></div>
    </div>

    <footer>
        <p><strong>&copy; 2024 Money Game جميع الحقوق محفوظة</strong></p>
    </footer>

    <script type="module">
        import { initializeApp } from 'https://www.gstatic.com/firebasejs/10.14.0/firebase-app.js';
        import { getDatabase, ref, onValue, remove } from 'https://www.gstatic.com/firebasejs/10.14.0/firebase-database.js';

        const firebaseConfig = {
            apiKey: "AIzaSyCnwJRJeedlD9cxua-Gbq0o5_vrbuHQNBw",
            authDomain: "moneygame-c9e55.firebaseapp.com",
            databaseURL: "https://moneygame-c9e55-default-rtdb.firebaseio.com",
            projectId: "moneygame-c9e55",
            storageBucket: "moneygame-c9e55.appspot.com",
            messagingSenderId: "589617824375",
            appId: "1:589617824375:web:44b8b8b944b5a577753c72",
            measurementId: "G-HL08MW00PB"
        };

        const app = initializeApp(firebaseConfig);
        const database = getDatabase(app);
        const registrantsRef = ref(database, 'registrants/');
        const approvedRef = ref(database, 'approvedRegistrants');

        // عرض قائمة المستخدمين
        onValue(registrantsRef, (snapshot) => {
            const users = snapshot.val();
            const userList = document.getElementById('userList');
            userList.innerHTML = '';

            if (users) {
                Object.keys(users).forEach(key => {
                    const user = users[key];
                    const userItem = document.createElement('div');
                    userItem.className = 'user-item';
                    userItem.innerHTML = `
                        <span>${user.name} (${user.email})</span>
                        <span class="remove-btn" onclick="removeUser('${key}', this)">×</span>
                    `;
                    userList.appendChild(userItem);
                });
            } else {
                userList.innerHTML = '<p>لا توجد مستخدمين.</p>';
            }
        });

        // وظيفة حذف المستخدم
        window.removeUser = function(userId, element) {
            if (confirm('هل أنت متأكد أنك تريد حذف هذا المستخدم؟')) {
                remove(ref(database, 'registrants/' + userId))
                    .then(() => {
                        alert('تم حذف المستخدم بنجاح!');
                        const userItem = element.parentElement;
                        userItem.remove();
                    })
                    .catch((error) => {
                        console.error("Error removing user: ", error);
                    });
            }
        };

        // تحميل المسجلين المعتمدين
        let approvedRegistrants = [];
        const displayLimit = 100;

        function loadApprovedRegistrants() {
            const loadingIndicator = document.getElementById('loadingIndicator');
            loadingIndicator.style.display = 'block';

            onValue(approvedRef, (snapshot) => {
                approvedRegistrants = snapshot.val() || {};
                displayRegistrants(approvedRegistrants);
                loadingIndicator.style.display = 'none';
            });
        }

        function displayRegistrants(registrants) {
            const approvedRegistrantsList = document.getElementById('approvedRegistrantsList');
            approvedRegistrantsList.innerHTML = '';

            const registrantKeys = Object.keys(registrants);
            let count = 0;

            registrantKeys.forEach((key, index) => {
                if (index < displayLimit) {
                    const registrant = registrants[key];
                    const box = document.createElement('div');
                    box.className = 'box';
                    box.innerHTML = `${registrant.name} <span class="remove-btn" onclick="removeApprovedUser('${key}', this)">×</span>`;
                    approvedRegistrantsList.appendChild(box);
                    count++;

                    if (count % 5 === 0) {
                        const br = document.createElement('div');
                        br.style.width = '100%';
                        approvedRegistrantsList.appendChild(br);
                    }
                }

                if (count === displayLimit) {
                    const waitingLine = document.createElement('div');
                    waitingLine.className = 'waiting-line';
                    waitingLine.innerHTML = '100 شخص في الانتظار';
                    approvedRegistrantsList.appendChild(waitingLine);
                }
            });

            const resultCount = document.getElementById('resultCount');
            resultCount.innerHTML = `عدد المشتركين: ${count}`;
        }

        window.removeApprovedUser = function(userId, element) {
            if (confirm('هل أنت متأكد أنك تريد حذف هذا المشترك؟')) {
                remove(ref(database, 'approvedRegistrants/' + userId))
                    .then(() => {
                        alert('تم حذف المشترك بنجاح!');
                        const boxItem = element.parentElement.parentElement;
                        boxItem.remove();
                    })
                    .catch((error) => {
                        console.error("Error removing approved user: ", error);
                    });
            }
        };

        function filterNames() {
            const searchInput = document.getElementById('searchInput');
            const filter = searchInput.value.toLowerCase();
            
            const filteredRegistrants = Object.keys(approvedRegistrants).filter(key => 
                approvedRegistrants[key].name.toLowerCase().includes(filter)
            );

            const approvedRegistrantsList = document.getElementById('approvedRegistrantsList');
            approvedRegistrantsList.innerHTML = '';

            filteredRegistrants.forEach((key, index) => {
                const registrant = approvedRegistrants[key];
                const box = document.createElement('div');
                box.className = 'box';
                box.innerHTML = `${registrant.name} <span class="remove-btn" onclick="removeApprovedUser('${key}', this)">×</span>`;
                approvedRegistrantsList.appendChild(box);
                
                if ((index + 1) % 5 === 0) {
                    const br = document.createElement('div');
                    br.style.width = '100%';
                    approvedRegistrantsList.appendChild(br);
                }
            });
        }

        document.addEventListener('DOMContentLoaded', () => {
            loadApprovedRegistrants();
            document.getElementById('searchInput').addEventListener('input', filterNames);
        });
    </script>
</body>
</html>
