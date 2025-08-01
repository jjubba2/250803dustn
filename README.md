# 250803dustn
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>숙제 제출 확인기</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: #f6f8fc;
      padding: 40px;
      max-width: 900px;
      margin: auto;
    }

    h1 {
      text-align: center;
      color: #1d3557;
    }

    h3 {
      margin-bottom: 10px;
      color: #457b9d;
    }

    .section {
      background-color: #ffffff;
      border-radius: 10px;
      padding: 20px;
      margin-bottom: 30px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.05);
    }

    input[type="text"] {
      padding: 10px;
      border: 1px solid #ccc;
      border-radius: 8px;
      width: 250px;
      font-size: 16px;
    }

    button {
      padding: 10px 15px;
      background-color: #457b9d;
      color: white;
      border: none;
      border-radius: 8px;
      font-size: 14px;
      cursor: pointer;
      margin-left: 5px;
      transition: 0.2s;
    }

    button:hover {
      background-color: #1d3557;
    }

    .tabs {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin-bottom: 20px;
    }

    .tab-wrapper {
      display: flex;
      align-items: center;
      background-color: #e0ecf9;
      padding: 8px 12px;
      border-radius: 8px;
    }

    .tab-btn {
      background-color: transparent;
      border: none;
      cursor: pointer;
      font-weight: bold;
      color: #1d3557;
    }

    .tab-btn.active {
      text-decoration: underline;
    }

    .delete-tab-btn {
      background-color: #e63946;
      color: white;
      border: none;
      border-radius: 50%;
      width: 20px;
      height: 20px;
      font-size: 14px;
      margin-left: 5px;
      cursor: pointer;
    }

    .tab-content {
      display: none;
      background-color: #ffffff;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.05);
    }

    .tab-content.active {
      display: block;
    }

    .student-row {
      display: flex;
      align-items: center;
      padding: 8px 10px;
      border-bottom: 1px solid #f0f0f0;
      border-radius: 5px;
      transition: background-color 0.2s;
    }

    .student-row.checked {
      background-color: #d0e8ff;
    }

    .student-row label {
      margin-left: 10px;
      font-size: 16px;
    }
  </style>
</head>
<body>

  <h1>📘 숙제 제출 확인기</h1>

  <div class="section">
    <h3>1️⃣ 우리 반 학생 명단 입력</h3>
    <input type="text" id="studentListInput" placeholder="예: 민수, 지은, 태오, 수빈">
    <button id="saveStudentListBtn">명단 저장</button>
  </div>

  <div class="section">
    <h3>2️⃣ 새 숙제 추가</h3>
    <input type="text" id="tabNameInput" placeholder="숙제 제목 입력">
    <button id="addTabBtn">숙제 추가</button>
  </div>

  <div class="tabs" id="tabButtons"></div>
  <div id="tabContents"></div>

  <script>
    const tabButtons = document.getElementById('tabButtons');
    const tabContents = document.getElementById('tabContents');
    const addTabBtn = document.getElementById('addTabBtn');
    const tabNameInput = document.getElementById('tabNameInput');
    const studentListInput = document.getElementById('studentListInput');
    const saveStudentListBtn = document.getElementById('saveStudentListBtn');

    let tabCount = 0;
    let studentNames = [];
    let homeworkData = JSON.parse(localStorage.getItem('homeworkData')) || { students: [], tabs: [] };

    // 초기화 시 불러오기
    function initFromStorage() {
      if (homeworkData.students.length > 0) {
        studentNames = homeworkData.students;
        studentListInput.value = studentNames.join(', ');
      }
      homeworkData.tabs.forEach((tab, index) => {
        addTab(tab.name, tab.checks);
      });
    }

    // 저장 함수
    function saveToStorage() {
      localStorage.setItem('homeworkData', JSON.stringify({
        students: studentNames,
        tabs: homeworkData.tabs
      }));
    }

    saveStudentListBtn.addEventListener('click', () => {
      const raw = studentListInput.value.trim();
      if (raw === '') return alert("학생 이름을 입력해주세요!");
      studentNames = raw.split(',').map(name => name.trim()).filter(name => name !== '');
      homeworkData.students = studentNames;
      saveToStorage();
      alert("학생 명단이 저장되었습니다!");
    });

    addTabBtn.addEventListener('click', () => {
      const tabName = tabNameInput.value.trim();
      if (tabName === '') return;
      if (studentNames.length === 0) {
        return alert("먼저 학생 명단을 저장해주세요.");
      }
      const checks = Array(studentNames.length).fill(false);
      homeworkData.tabs.push({ name: tabName, checks });
      saveToStorage();
      addTab(tabName, checks);
      tabNameInput.value = '';
    });

    function addTab(tabName, checks) {
      const tabId = 'tab' + tabCount++;

      const tabBtn = document.createElement('button');
      tabBtn.textContent = tabName;
      tabBtn.className = 'tab-btn';
      tabBtn.dataset.tab = tabId;

      const deleteBtn = document.createElement('button');
      deleteBtn.textContent = '×';
      deleteBtn.className = 'delete-tab-btn';

      const wrapper = document.createElement('div');
      wrapper.className = 'tab-wrapper';
      wrapper.appendChild(tabBtn);
      wrapper.appendChild(deleteBtn);
      tabButtons.appendChild(wrapper);

      tabBtn.addEventListener('click', () => {
        document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
        document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
        tabBtn.classList.add('active');
        document.getElementById(tabId).classList.add('active');
      });

      deleteBtn.addEventListener('click', () => {
        if (confirm(`'${tabName}' 숙제를 삭제하시겠습니까?`)) {
          wrapper.remove();
          document.getElementById(tabId).remove();
          homeworkData.tabs = homeworkData.tabs.filter(tab => tab.name !== tabName);
          saveToStorage();
        }
      });

      const tabContent = document.createElement('div');
      tabContent.className = 'tab-content';
      tabContent.id = tabId;

      const studentList = document.createElement('div');

      studentNames.forEach((name, idx) => {
        const row = document.createElement('div');
        row.className = 'student-row';

        const checkbox = document.createElement('input');
        checkbox.type = 'checkbox';
        checkbox.checked = checks[idx];
        if (checkbox.checked) row.classList.add('checked');

        const label = document.createElement('label');
        label.textContent = name;

        checkbox.addEventListener('change', () => {
          row.classList.toggle('checked', checkbox.checked);
          checks[idx] = checkbox.checked;
          saveToStorage();
        });

        row.appendChild(checkbox);
        row.appendChild(label);
        studentList.appendChild(row);
      });

      tabContent.appendChild(studentList);
      tabContents.appendChild(tabContent);

      tabBtn.click();
    }

    initFromStorage();
  </script>

</body>
</html>
