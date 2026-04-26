<!DOCTYPE html>
<html lang="pt">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Save 2024 - Compartilhamento Público</title>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore-compat.js"></script>
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
            transition: all 0.3s ease;
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
            transition: transform 0.2s;
        }
        .file-card:hover { transform: translateY(-5px); }
        .file-preview { width: 100%; height: 200px; object-fit: cover; }
        .file-info { padding: 15px; }
        .file-name { font-weight: bold; word-break: break-all; font-size: 0.9em; }
        .file-meta { color: #666; font-size: 0.7em; margin-top: 5px; }
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
        .progress-bar {
            width: 100%;
            height: 3px;
            background: #e0e0e0;
            margin-top: 10px;
            display: none;
        }
        .progress-fill {
            width: 0%;
            height: 100%;
            background: #667eea;
            transition: width 0.3s;
        }
        @media (max-width: 768px) {
            .files-grid { grid-template-columns: repeat(auto-fill, minmax(150px, 1fr)); }
            h1 { font-size: 1.8em; }
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>📱 Save 2024</h1>
            <p>Compartilhamento PÚBLICO - Todos veem os mesmos arquivos!</p>
            <div class="status">🌐 Conectado - Modo Público (Cloudinary)</div>
        </header>

        <div class="upload-area" onclick="document.getElementById('fileInput').click()">
            <div class="upload-icon">📤</div>
            <div>Clique para fazer upload (Todos verão)</div>
            <div style="font-size: 0.8em; margin-top: 10px;">📷 Fotos • 🎥 Vídeos • 📄 Documentos</div>
            <input type="file" id="fileInput" multiple accept="image/*,video/*,.pdf,.doc,.docx,.txt">
            <div class="progress-bar" id="progressBar">
                <div class="progress-fill" id="progressFill"></div>
            </div>
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
        // Firebase Config (já configurado)
        const firebaseConfig = {
            apiKey: "AIzaSyAw08LmN-SGTtWa61wk3QV",
            authDomain: "save2024.firebaseapp.com",
            projectId: "save2024",
            storageBucket: "save2024.firebasestorage.app",
            messagingSenderId: "702295359234",
            appId: "1:702295359234:web:ed5d0dbff4"
        };
        
        // Initialize Firebase
        firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();
        const auth = firebase.auth();
        
        // Cloudinary Config (SEU CLOUD NAME)
        const CLOUD_NAME = "dzm7bahp4";
        const UPLOAD_PRESET = "save2024"; // Vamos criar
        
        let currentFilter = 'all';
        
        // Login anônimo no Firebase
        auth.signInAnonymously().catch(error => console.error("Erro login:", error));
        
        // Função para enviar arquivo via Cloudinary
        async function uploadToCloudinary(file) {
            const formData = new FormData();
            formData.append('file', file);
            formData.append('upload_preset', 'save2024');
            formData.append('cloud_name', CLOUD_NAME);
            
            // Mostrar progresso
            const progressBar = document.getElementById('progressBar');
            const progressFill = document.getElementById('progressFill');
            progressBar.style.display = 'block';
            
            return new Promise((resolve, reject) => {
                const xhr = new XMLHttpRequest();
                xhr.open('POST', `https://api.cloudinary.com/v1_1/${CLOUD_NAME}/auto/upload`);
                xhr.upload.onprogress = (e) => {
                    if (e.lengthComputable) {
                        const percent = (e.loaded / e.total) * 100;
                        progressFill.style.width = percent + '%';
                    }
                };
                xhr.onload = () => {
                    progressBar.style.display = 'none';
                    progressFill.style.width = '0%';
                    if (xhr.status === 200) {
                        resolve(JSON.parse(xhr.responseText));
                    } else {
                        reject(new Error('Upload falhou'));
                    }
                };
                xhr.onerror = () => reject(new Error('Erro de rede'));
                xhr.send(formData);
            });
        }
        
        // Upload de arquivo
        async function uploadFile(file) {
            const fileType = file.type.startsWith('image/') ? 'image' : 
                           file.type.startsWith('video/') ? 'video' : 'document';
            
            const fileId = Date.now() + '_' + Math.random().toString(36).substr(2, 9);
            
            try {
                // Upload para Cloudinary
                const result = await uploadToCloudinary(file);
                
                // Salvar no Firestore
                await db.collection('files').doc(fileId).set({
                    id: fileId,
                    name: file.name,
                    type: fileType,
                    size: file.size,
                    url: result.secure_url,
                    publicId: result.public_id,
                    date: new Date().toISOString(),
                    uploader: auth.currentUser?.uid || 'anonimo'
                });
                
                alert('✅ Arquivo enviado para todos os usuários!');
            } catch(error) {
                alert('❌ Erro: ' + error.message);
            }
        }
        
        // Carregar arquivos do Firestore
        function loadFiles() {
            db.collection('files').orderBy('date', 'desc').onSnapshot(snapshot => {
                const files = [];
                snapshot.forEach(doc => {
                    files.push(doc.data());
                });
                displayFiles(files);
            }, error => {
                console.error("Erro:", error);
                document.getElementById('filesGrid').innerHTML = `
                    <div style="text-align:center;padding:40px;">
                        ⚠️ Erro ao carregar. Configure o Firestore no Firebase!
                    </div>
                `;
            });
        }
        
        // Exibir arquivos na tela
        function displayFiles(files) {
            const grid = document.getElementById('filesGrid');
            let filtered = files;
            if (currentFilter !== 'all') {
                filtered = files.filter(f => f.type === currentFilter);
            }
            
            if (filtered.length === 0) {
                grid.innerHTML = '<div style="grid-column:1/-1; text-align:center; padding:40px;">📂 Nenhum arquivo público. Envie o primeiro!</div>';
                return;
            }
            
            grid.innerHTML = filtered.map(file => `
                <div class="file-card" onclick="window.open('${file.url}', '_blank')">
                    ${file.type === 'image' ? 
                        `<img class="file-preview" src="${file.url}" alt="${file.name}">` :
                      file.type === 'video' ? 
                        `<video class="file-preview" src="${file.url}"></video>` :
                        `<div class="file-preview" style="display:flex;align-items:center;justify-content:center;font-size:48px;background:#f0f0f0;">📄</div>`
                    }
                    <div class="file-info">
                        <div class="file-name">${file.name.length > 30 ? file.name.substring(0,27)+'...' : file.name}</div>
                        <div class="file-meta">${formatFileSize(file.size)} • ${new Date(file.date).toLocaleString('pt-BR')}</div>
                        <button class="delete-btn" onclick="event.stopPropagation(); deleteFile('${file.id}', '${file.publicId}')">🗑️ Excluir</button>
                    </div>
                </div>
            `).join('');
        }
        
        // Deletar arquivo
        async function deleteFile(fileId, publicId) {
            if (!confirm('⚠️ Excluir este arquivo para TODOS os usuários?')) return;
            
            try {
                await db.collection('files').doc(fileId).delete();
                alert('✅ Arquivo removido para todos!');
            } catch(error) {
                alert('❌ Erro: ' + error.message);
            }
        }
        
        // Formatar tamanho
        function formatFileSize(bytes) {
            if (bytes === 0) return '0 Bytes';
            const k = 1024;
            const sizes = ['Bytes', 'KB', 'MB', 'GB'];
            const i = Math.floor(Math.log(bytes) / Math.log(k));
            return parseFloat((bytes / Math.pow(k, i)).toFixed(1)) + ' ' + sizes[i];
        }
        
        // Filtrar arquivos
        function filterFiles(type) {
            currentFilter = type;
            loadFiles();
            document.querySelectorAll('.filter-btn').forEach(btn => btn.classList.remove('active'));
            event.target.classList.add('active');
        }
        
        // Fechar modal
        function closeModal() {
            document.getElementById('modal').style.display = 'none';
            document.getElementById('modalContent').innerHTML = '';
        }
        
        // Eventos de upload
        document.getElementById('fileInput').addEventListener('change', (e) => {
            const files = Array.from(e.target.files);
            files.forEach(uploadFile);
            e.target.value = '';
        });
        
        // Drag and drop
        const uploadArea = document.querySelector('.upload-area');
        uploadArea.addEventListener('dragover', (e) => {
            e.preventDefault();
            uploadArea.style.background = 'linear-gradient(135deg, #e8eaf6 0%, #b39ddb 100%)';
        });
        uploadArea.addEventListener('dragleave', (e) => {
            e.preventDefault();
            uploadArea.style.background = 'linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%)';
        });
        uploadArea.addEventListener('drop', (e) => {
            e.preventDefault();
            uploadArea.style.background = 'linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%)';
            const files = Array.from(e.dataTransfer.files);
            files.forEach(uploadFile);
        });
        
        // Iniciar
        loadFiles();
    </script>
</body>
</html>                        e.
