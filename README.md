# Cardinal-dashboard-mvp1
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Cardinal Landscaping Dashboard</title>
<style>
body { font-family: Arial, sans-serif; margin:0; padding:0; background:#f5f5f5; }
header { background:#2E7D32; color:white; padding:20px; text-align:center; }
.container { padding:20px; }
section { background:white; padding:20px; margin-bottom:15px; border-radius:8px; box-shadow:0 2px 5px rgba(0,0,0,0.1); }
h2 { margin-top:0; color:#2E7D32; cursor:pointer; }
ul { list-style:none; padding-left:0; display:none; }
li { padding:5px 0; border-radius:4px; display:flex; justify-content: space-between; align-items:center; }
li.urgent { background:#FFCDD2; font-weight:bold; }
li.new { background:#C8E6C9; }
li.today { background:#FFECB3; }
li.completed { color:#999; text-decoration: line-through; }
li.new-highlight { animation: flashHighlight 2s ease; }
@keyframes flashHighlight {
  0% { background-color: #C8E6C9; }
  50% { background-color: #fff59d; }
  100% { background-color: #C8E6C9; }
}
.badge { padding:2px 6px; border-radius:4px; color:white; font-size:12px; margin-right:5px; }
.badge.Appointments { background:green; }
.badge.Messages { background:blue; }
.badge.Projects { background:orange; }
button.action { margin-left:5px; padding:2px 6px; border:none; border-radius:4px; cursor:pointer; }
button.complete { background:#4CAF50; color:white; }
button.delete { background:#F44336; color:white; }
a.form-link, button.export { display:inline-block; margin-top:10px; padding:8px 12px; background:#2E7D32; color:white; border-radius:5px; text-decoration:none; border:none; cursor:pointer; }
a.form-link:hover, button.export:hover { background:#1B5E20; }
label { margin-right:10px; }
input, select { padding:5px; margin-bottom:10px; width:100%; box-sizing:border-box; }
.filter-bar { display:flex; flex-wrap: wrap; gap:10px; margin-bottom:10px; }
#searchInput { flex:1; padding:5px; }
@media (max-width:600px) {
    .filter-bar { flex-direction: column; }
    li { flex-direction: column; align-items: flex-start; }
    button.action { margin-left: 0; margin-top: 5px; }
}
</style>
</head>
<body>

<header>
<h1>Cardinal Landscaping Dashboard</h1>
</header>

<div class="container">

<!-- Filter/Search -->
<div class="filter-bar">
<input type="text" id="searchInput" placeholder="Search...">
<label>Status:
<select id="statusFilter">
<option value="">All</option>
<option value="Pending">Pending</option>
<option value="Completed">Completed</option>
</select>
</label>
<label>Urgent:
<select id="urgentFilter">
<option value="">All</option>
<option value="Yes">Yes</option>
<option value="No">No</option>
</select>
</label>
<button class="export" id="exportBtn">Export CSV</button>
</div>

<!-- Add Item Form -->
<section>
<h2 onclick="toggleSection('addItemFormSection')">Add New Item ▼</h2>
<div id="addItemFormSection" style="display:none;">
<form id="addItemForm">
<label>Type:
<select id="itemType">
<option value="Appointments">Appointment</option>
<option value="Messages">Message</option>
<option value="Projects">Project</option>
</select>
</label>
<label>Text: <input type="text" id="itemText" required></label>
<label>Date: <input type="date" id="itemDate"></label>
<label>Status:
<select id="itemStatus">
<option value="">Pending</option>
<option value="Completed">Completed</option>
</select>
</label>
<label>Urgent:
<select id="itemUrgent">
<option value="">No</option>
<option value="Yes">Yes</option>
</select>
</label>
<button type="submit">Add Item</button>
</form>
</div>
</section>

<!-- Sections -->
<section>
<h2 onclick="toggleSection('appointmentsList')">Upcoming Appointments ▼</h2>
<ul id="appointmentsList">Loading...</ul>
</section>

<section>
<h2 onclick="toggleSection('messagesList')">Client Messages ▼</h2>
<ul id="messagesList">Loading...</ul>
</section>

<section>
<h2 onclick="toggleSection('projectsList')">Ongoing Projects ▼</h2>
<ul id="projectsList">Loading...</ul>
</section>

</div>

<script>
// Toggle section
function toggleSection(id) {
  const el = document.getElementById(id);
  el.style.display = el.style.display === "none" ? "block" : "none";
}

// Sheets & lists
const sheetId = "1_hCWSkplen_HTtVLsHOXoP-3xHqebCCIk6_adTaUua8";
const tabs = ["Appointments", "Messages", "Projects"];
const lists = ["appointmentsList", "messagesList", "projectsList"];
let allData = {};

// Load data
function loadData() {
  tabs.forEach((tab, index) => {
    fetch(`https://opensheet.vercel.app/${sheetId}/${tab}`)
    .then(res => res.json())
    .then(data => {
      allData[tab] = data;
      renderList(tab, index);
      // Check for urgent/today alerts
      const todayStr = new Date().toISOString().split('T')[0];
      data.forEach(item => {
        if(item.Urgent === 'Yes' || item.Date === todayStr){
            console.log(`⚠️ Alert: ${item.Text} is urgent or due today.`);
            // Optional: alert(`⚠️ Alert: ${item.Text} is urgent or due today.`);
        }
      });
    })
    .catch(err => console.error(err));
  });
}

// Render with filters/search and sorting
function renderList(tab, index) {
  const ul = document.getElementById(lists[index]);
  ul.innerHTML = "";
  const statusFilter = document.getElementById("statusFilter").value;
  const urgentFilter = document.getElementById("urgentFilter").value;
  const search = document.getElementById("searchInput").value.toLowerCase();
  const todayStr = new Date().toISOString().split('T')[0];
  const now = new Date();

  // Sort by date ascending
  allData[tab].sort((a,b)=>new Date(a.Date || 0) - new Date(b.Date || 0));

  allData[tab].forEach(item => {
    if(statusFilter && item.Status !== statusFilter) return;
    if(urgentFilter && item.Urgent !== urgentFilter) return;
    if(search && !item.Text.toLowerCase().includes(search)) return;

    const li = document.createElement('li');

    const leftDiv = document.createElement('div');
    const badge = document.createElement('span');
    badge.className = `badge ${tab}`;
    badge.textContent = tab;
    leftDiv.appendChild(badge);
    const textSpan = document.createElement('span');
    const dateText = item.Date ? ` (${item.Date})` : '';
    textSpan.textContent = item.Text + dateText;
    leftDiv.appendChild(textSpan);
    li.appendChild(leftDiv);

    const btnDiv = document.createElement('div');
    const completeBtn = document.createElement('button');
    completeBtn.className = "action complete";
    completeBtn.textContent = "✅";
    completeBtn.onclick = () => { li.classList.toggle('completed'); item.Status = 'Completed'; };
    const deleteBtn = document.createElement('button');
    deleteBtn.className = "action delete";
    deleteBtn.textContent = "🗑️";
    deleteBtn.onclick = () => { li.remove(); };
    btnDiv.appendChild(completeBtn);
    btnDiv.appendChild(deleteBtn);
    li.appendChild(btnDiv);

    if(item.Status && item.Status.toLowerCase() === "completed") li.classList.add('completed');
    if(item.Urgent && item.Urgent.toLowerCase() === "yes") li.classList.add('urgent');
    if(item.Date && item.Date === todayStr) li.classList.add('today');
    if(item.Date && (now - new Date(item.Date))/36e5 <= 24) li.classList.add('new');

    ul.appendChild(li);
  });
  ul.style.display = "block";
}

// Event listeners for filters/search
document.getElementById("statusFilter").addEventListener("change", () => tabs.forEach((tab, i)=>renderList(tab,i)));
document.getElementById("urgentFilter").addEventListener("change", () => tabs.forEach((tab, i)=>renderList(tab,i)));
document.getElementById("searchInput").addEventListener("input", () => tabs.forEach((tab, i)=>renderList(tab,i)));

// Export CSV
document.getElementById("exportBtn").addEventListener("click", () => {
  let csvContent = "data:text/csv;charset=utf-8,Type,Text,Date,Status,Urgent\n";
  tabs.forEach(tab => {
    allData[tab].forEach(item => {
      csvContent += `${tab},"${item.Text}",${item.Date || ""},${item.Status || ""},${item.Urgent || ""}\n`;
    });
  });
  const encodedUri = encodeURI(csvContent);
  const link = document.createElement("a");
  link.setAttribute("href", encodedUri);
  link.setAttribute("download", "dashboard_export.csv");
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
});

// Add Item form
document.getElementById("addItemForm").addEventListener("submit", function(e){
  e.preventDefault();
  const type = document.getElementById("itemType").value;
  const text = document.getElementById("itemText").value;
  const date = document.getElementById("itemDate").value;
  const status = document.getElementById("itemStatus").value;
  const urgent = document.getElementById("itemUrgent").value;

  const newItem = { Text: text, Date: date, Status: status, Urgent: urgent };
  allData[type].unshift(newItem);

  // Render and highlight new
  const index = tabs.indexOf(type);
  renderList(type, index);
  const ul = document.getElementById(lists[index]);
  if(ul.firstChild) {
      ul.firstChild.classList.add('new-highlight');
      setTimeout(()=>ul.firstChild.classList.remove('new-highlight'),2000);
  }

  // Reset form
  document.getElementById("addItemForm").reset();
});

// Initial load and auto-refresh every 30s
loadData();
setInterval(loadData, 30000);
</script>

</body>
</html>
