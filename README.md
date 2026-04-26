# Save2024
APP para compartilhar videos fotos videos e documentos 
<!DOCTYPE html>
<html lang="pt">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Save 2024 - Compartilhamento Público</title>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-storage-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-auth-compat.js"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: rgba(255,255,255,0.95);
            border-radius: 20px;
            padding: 20px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
        }
        header { text-align: center; margin-bottom: 30px; }
        h1 { font-size: 2.5em; color: #667eea; }
        .upload-area {
            background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            border: 3px dashed #667eea;
            border-radius: 15px;
            padding: 40px;
            text-align: center;
            cursor: pointer;
            margin-bottom: 30px;
        }
        .upload-area:hover { background: linear-gradient(135deg, #e8eaf6 0%, #b39ddb 100%); }
        .upload-icon { font-size: 48px; }
        #fileInput { display: none; }
        .filters { display: flex; gap: 10px; margin-bottom: 20px; flex-wrap: wrap; }
        .filter-btn {
            padding: 10px 20px;
            background: #f0f0f0;
            border: none;
            border-radius: 25px;
            cursor: pointer;
        }
        .filter-btn.active { background: #667eea; color: white; }
        .files-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
            gap: 20px;
        }
        .file-card {
            background: white;
            border-radius: 10px;
            overflow: hidden;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            cursor: pointer;
        }
        .file-preview { width: 100%; height: 200px; object-fit: cover; }
        .file-info { padding: 15px; }
        .file-name { font-weight: bold; word-break: break-all; }
        .delete-btn {
            background: #f44336;
            color: white;
            border: none;
            padding: 5px 10px;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 10px;
            width: 100%;
        }
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.9);
            z-index: 1000;
            justify-content: center;
            align-items: center;
        }
        .modal-close {
            position: absolute;
            top: 20px;
            right: 40px;
            color: white;
            font-size: 40px;
            cursor: pointer;
        }
        footer { text-align: center; margin-top: 30px; padding-top: 20px; color: #666; }
        .credits { background: linear-gradient(135deg, #667eea, #764ba2); -webkit-background-clip: text; background-clip: text; color: transparent; font-weight: bold; }
        .status { background: #4caf50; color: white; padding: 5px 10px; border-radius: 20px; display: inline-block; margin-bottom: 15px; font-size: 0.8em; }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>📱 Save 2024</h1>
            <p>Compartilhamento PÚBLICO - Todos veem os mesmos arquivos!</p>
            <div class="status">🌐 Conectado - Modo Público</div>
        </header>

        <div class="upload-area" onclick="document.getElementById('fileInput').click()">
            <div class="upload-icon">📤</div>
            <div>Clique para fazer upload (Todos verão)</div>
            <input type="file" id="fileInput" multiple accept="image/*,video/*,.pdf,.doc,.txt">
        </div>

        <div class="filters">
            <button class="filter-btn active" onclick="filterFiles('all')">Todos</button>
            <button class="filter-btn" onclick="filterFiles('image')">📷 Fotos</button>
            <button class="filter-btn" onclick="filterFiles('video')">🎥 Vídeos</button>
            <button class="filter-btn" onclick="filterFiles('document')">📄 Documentos</button>
        </div>

        <div id="filesGrid" class="files-grid">
            <div style="text-align:center; padding:40px;">📂 Carregando arquivos públicos...</div>
        </div>

        <footer>
            <p>Criado por <span class="credits">Chalito Martins José Chale</span> © 2026</p>
            <p>✅ Arquivos públicos - Qualquer pessoa pode ver o que você enviar!</p>
        </footer>
    </div>

    <div id="modal" class="modal" onclick="closeModal()">
        <span class="modal-close">&times;</span>
        <div class="modal-content" id="modalContent"></div>
    </div>

    <script>
        const firebaseConfig = {
            apiKey: "AIzaSyAw08LmN-SGTtWa61wk3QV",
            authDomain: "save2024.firebaseapp.com",
            projectId: "save2024",
            storageBucket: "save2024.firebasestorage.googleapis.com",
            messagingSenderId: "702295359234",
            appId: "1:702295359234:web:ed5d0dbff4"
        };
        
        firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();
        const storage = firebase.storage();
        const auth = firebase.auth();
        
        auth.signInAnonymously().catch(error => console.error("Erro:", error));
        
        let currentFilter = 'all';
        
        async function uploadFile(file) {
            const fileId = Date.now() + '_' + Math.random().toString(36).substr(2, 9);
            const fileType = file.type.startsWith('image/') ? 'image' : file.type.startsWith('video/') ? 'video' : 'document';
            
            try {
                const storageRef = storage.ref(`uploads/${fileId}`);
                await storageRef.put(file);
                const url = await storageRef.getDownloadURL();
                
                await db.collection('files').doc(fileId).set({
                    id: fileId,
                    name: file.name,
                    type: fileType,
                    size: file.size,
                    url: url,
                    date: new Date().toISOString()
                });
                alert('✅ Arquivo enviado para todos!');
            } catch(error) {
                alert('❌ Erro: ' + error.message);
            }
        }
        
        function loadFiles() {
            db.collection('files').orderBy('date', 'desc').onSnapshot(snapshot => {
                const files = [];
                snapshot.forEach(doc => files.push(doc.data()));
                displayFiles(files);
            });
        }
        
        function displayFiles(files) {
            const grid = document.getElementById('filesGrid');
            let filtered = files;
            if (currentFilter !== 'all') filtered = files.filter(f => f.type === currentFilter);
            
            if (filtered.length === 0) {
                grid.innerHTML = '<div style="grid-column:1/-1; text-align:center; padding:40px;">📂 Nenhum arquivo público. Envie o primeiro!</div>';
                return;
            }
            
            grid.innerHTML = filtered.map(file => `
                <div class="file-card" onclick="window.open('${file.url}', '_blank')">
                    ${file.type === 'image' ? `<img class="file-preview" src="${file.url}">` :
                      file.type === 'video' ? `<video class="file-preview" src="${file.url}"></video>` :
                      `<div class="file-preview" style="display:flex;align-items:center;justify-content:center;font-size:48px;">📄</div>`}
                    <div class="file-info">
                        <div class="file-name">${file.name}</div>
                        <div class="file-size">${(file.size/1024/1024).toFixed(2)} MB</div>
                        <button class="delete-btn" onclick="event.stopPropagation(); deleteFile('${file.id}')">🗑️ Excluir</button>
                    </div>
                </div>
            `).join('');
        }
        
        async function deleteFile(fileId) {
            if (!confirm('Excluir para todos os usuários?')) return;
            await db.collection('files').doc(fileId).delete();
            await storage.ref(`uploads/${fileId}`).delete().catch(() => {});
            alert('✅ Removido para todos!');
        }
        
        function filterFiles(type) {
            currentFilter = type;
            loadFiles();
            document.querySelectorAll('.filter-btn').forEach(btn => btn.classList.remove('active'));
            event.target.classList.add('active');
        }
        
        function closeModal() {
            document.getElementById('modal').style.display = 'none';
        }
        
        document.getElementById('fileInput').addEventListener('change', (e) => {
            Array.from(e.target.files).forEach(uploadFile);
            e.target.value = '';
        });
        
        document.querySelector('.upload-area').addEventListener('dragover', e => e.preventDefault());
        document.querySelector('.upload-area').addEventListener('drop', e => {
            e.preventDefault();
            Array.from(e.dataTransfer.files).forEach(uploadFile);
        });
        
        loadFiles();
    </script>
</body>
</html>
