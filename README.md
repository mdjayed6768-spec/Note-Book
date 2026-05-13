# Note-Book
A simple notebook app to save notes online.
<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Notebook - Jayed</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            background: #f0f2f5;
            color: #333;
            transition: 0.4s;
        }
        body.dark {
            background: #121212;
            color: #e0e0e0;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 15px;
        }
        header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 15px 0;
            border-bottom: 2px solid #ddd;
        }
        body.dark header { border-color: #333; }
        h1 { font-size: 2.2rem; }
        .controls {
            display: flex;
            gap: 10px;
            align-items: center;
        }
        input, button, textarea {
            padding: 10px 15px;
            border: none;
            border-radius: 8px;
        }
        input[type="text"] {
            width: 280px;
            background: white;
            border: 1px solid #ccc;
        }
        body.dark input[type="text"] { background: #2a2a2a; border-color: #444; color: white; }
        button {
            background: #667eea;
            color: white;
            cursor: pointer;
            font-weight: bold;
        }
        button:hover { background: #5a6fd8; }
        .new-btn {
            background: #4caf50;
            padding: 12px 20px;
        }
        .note-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
            gap: 15px;
            margin-top: 20px;
        }
        .note {
            background: white;
            padding: 18px;
            border-radius: 12px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.1);
            transition: 0.3s;
            cursor: pointer;
        }
        body.dark .note {
            background: #1e1e1e;
            box-shadow: 0 4px 12px rgba(0,0,0,0.4);
        }
        .note:hover { transform: translateY(-5px); }
        .note-title {
            font-size: 1.3rem;
            margin-bottom: 8px;
            font-weight: 600;
        }
        .note-date {
            font-size: 0.85rem;
            color: #888;
            margin-bottom: 10px;
        }
        .note-content {
            display: -webkit-box;
            -webkit-line-clamp: 4;
            -webkit-box-orient: vertical;
            overflow: hidden;
        }
        .modal {
            display: none;
            position: fixed;
            top: 0; left: 0;
            width: 100%; height: 100%;
            background: rgba(0,0,0,0.7);
            z-index: 1000;
            align-items: center;
            justify-content: center;
        }
        .modal-content {
            background: white;
            width: 90%;
            max-width: 700px;
            border-radius: 12px;
            padding: 20px;
        }
        body.dark .modal-content { background: #1e1e1e; }
        textarea {
            width: 100%;
            height: 300px;
            resize: vertical;
            margin: 15px 0;
            font-size: 1.1rem;
        }
        .delete-btn {
            background: #f44336;
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>📒 My Notebook</h1>
            <div class="controls">
                <input type="text" id="search" placeholder="নোট খুঁজুন...">
                <button class="new-btn" onclick="newNote()">+ নতুন নোট</button>
                <button onclick="toggleDarkMode()">🌙</button>
            </div>
        </header>

        <div class="note-grid" id="noteGrid"></div>
    </div>

    <!-- Modal -->
    <div class="modal" id="noteModal">
        <div class="modal-content">
            <input type="text" id="noteTitle" placeholder="নোটের শিরোনাম" style="width:100%; font-size:1.4rem; margin-bottom:10px;">
            <textarea id="noteText" placeholder="এখানে লিখুন..."></textarea>
            <div style="display:flex; gap:10px; justify-content:end;">
                <button class="delete-btn" onclick="deleteCurrentNote()">ডিলিট</button>
                <button onclick="closeModal()">বাতিল</button>
                <button onclick="saveNote()">সেভ করুন</button>
            </div>
        </div>
    </div>

    <script>
        let notes = JSON.parse(localStorage.getItem('notes')) || [];
        let currentEditId = null;

        function saveToLocal() {
            localStorage.setItem('notes', JSON.stringify(notes));
        }

        function renderNotes(filteredNotes = notes) {
            const grid = document.getElementById('noteGrid');
            grid.innerHTML = '';

            if (filteredNotes.length === 0) {
                grid.innerHTML = `<p style="text-align:center; grid-column:1/-1; padding:50px; color:#888;">কোনো নোট পাওয়া যায়নি 😔</p>`;
                return;
            }

            filteredNotes.forEach((note, index) => {
                const div = document.createElement('div');
                div.className = 'note';
                div.innerHTML = `
                    <div class="note-title">${note.title || 'Untitled'}</div>
                    <div class="note-date">${new Date(note.date).toLocaleDateString('bn-BD')} ${new Date(note.date).toLocaleTimeString('bn-BD', {hour:'2-digit', minute:'2-digit'})}</div>
                    <div class="note-content">\( {note.text.substring(0, 150)} \){note.text.length > 150 ? '...' : ''}</div>
                `;
                div.onclick = () => editNote(index);
                grid.appendChild(div);
            });
        }

        function newNote() {
            currentEditId = null;
            document.getElementById('noteTitle').value = '';
            document.getElementById('noteText').value = '';
            document.getElementById('noteModal').style.display = 'flex';
        }

        function saveNote() {
            const title = document.getElementById('noteTitle').value.trim();
            const text = document.getElementById('noteText').value.trim();

            if (!text) {
                alert("নোটে কিছু লিখুন!");
                return;
            }

            if (currentEditId !== null) {
                notes[currentEditId] = {
                    title: title || 'Untitled',
                    text: text,
                    date: Date.now()
                };
            } else {
                notes.unshift({
                    title: title || 'Untitled',
                    text: text,
                    date: Date.now()
                });
            }

            saveToLocal();
            closeModal();
            renderNotes();
        }

        function editNote(index) {
            currentEditId = index;
            const note = notes[index];
            document.getElementById('noteTitle').value = note.title;
            document.getElementById('noteText').value = note.text;
            document.getElementById('noteModal').style.display = 'flex';
        }

        function deleteCurrentNote() {
            if (currentEditId !== null && confirm("এই নোট ডিলিট করবেন?")) {
                notes.splice(currentEditId, 1);
                saveToLocal();
                closeModal();
                renderNotes();
            }
        }

        function closeModal() {
            document.getElementById('noteModal').style.display = 'none';
        }

        function toggleDarkMode() {
            document.body.classList.toggle('dark');
        }

        // Search Functionality
        document.getElementById('search').addEventListener('input', (e) => {
            const term = e.target.value.toLowerCase();
            const filtered = notes.filter(note => 
                note.title.toLowerCase().includes(term) || 
                note.text.toLowerCase().includes(term)
            );
            renderNotes(filtered);
        });

        // Initialize
        renderNotes();
    </script>
</body>
</html>
