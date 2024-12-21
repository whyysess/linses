<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Folder View with File Open (Like Google Drive)</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #0d1117;
            color: #ffffff;
            text-align: center;
            padding: 20px;
        }
        h1 {
            color: #00ff88;
        }
        #fileInput {
            margin: 20px;
            padding: 10px;
            background-color: #00ff88;
            color: #0d1117;
            border: none;
            border-radius: 8px;
            cursor: pointer;
        }
        .file-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }
        .file-item {
            text-align: center;
            background-color: #1a1d24;
            padding: 10px;
            border-radius: 10px;
            cursor: pointer;
            transition: transform 0.3s ease;
        }
        .file-item:hover {
            transform: scale(1.1);
        }
        .file-item img, .file-item video, .file-item audio {
            max-width: 100%;
            max-height: 150px;
            border-radius: 8px;
        }
        .file-item span {
            display: block;
            margin-top: 10px;
            font-size: 14px;
            color: #ffffff;
        }
        .file-viewer {
            display: none;
            background-color: rgba(0, 0, 0, 0.8);
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            justify-content: center;
            align-items: center;
            z-index: 10;
        }
        .file-viewer-content {
            background-color: #2e3338;
            padding: 20px;
            border-radius: 10px;
            max-width: 80%;
            max-height: 80%;
            overflow: auto;
        }
        .file-viewer-content img, .file-viewer-content video, .file-viewer-content audio {
            width: 100%;
            max-height: 600px;
        }
        .close-btn {
            position: absolute;
            top: 10px;
            right: 10px;
            padding: 10px;
            background-color: #ff5c5c;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        .close-btn:hover {
            background-color: #ff2f2f;
        }
    </style>
</head>
<body>
    <h1>Upload and View Files in Folder (Like Google Drive)</h1>
    <input type="file" id="fileInput" multiple accept="image/*,video/*,audio/*,application/pdf,.txt">
    <div class="file-grid" id="fileGrid"></div>

    <div id="fileViewer" class="file-viewer">
        <div class="file-viewer-content">
            <button class="close-btn" id="closeBtn">Close</button>
            <div id="fileContent"></div>
        </div>
    </div>

    <script>
        const fileInput = document.getElementById('fileInput');
        const fileGrid = document.getElementById('fileGrid');
        const fileViewer = document.getElementById('fileViewer');
        const fileContent = document.getElementById('fileContent');
        const closeBtn = document.getElementById('closeBtn');

        // Menyimpan dan menampilkan file yang diunggah
        function saveAndDisplayFiles(files) {
            Array.from(files).forEach(file => {
                const reader = new FileReader();

                reader.onload = function(event) {
                    const fileData = event.target.result;
                    const fileType = file.type;
                    const fileItem = document.createElement('div');
                    fileItem.classList.add('file-item');

                    let fileElement;
                    const fileName = file.name;

                    // Menyesuaikan tampilan berdasarkan tipe file
                    if (fileType.startsWith('image/')) {
                        fileElement = document.createElement('img');
                        fileElement.src = fileData;
                    } else if (fileType.startsWith('audio/')) {
                        fileElement = document.createElement('audio');
                        fileElement.controls = true;
                        fileElement.src = fileData;
                    } else if (fileType.startsWith('video/')) {
                        fileElement = document.createElement('video');
                        fileElement.controls = true;
                        fileElement.src = fileData;
                    } else if (fileType === 'application/pdf') {
                        fileElement = document.createElement('iframe');
                        fileElement.src = fileData;
                        fileElement.width = "100%";
                        fileElement.height = "400px";
                    } else if (fileType === 'text/plain') {
                        fileElement = document.createElement('pre');
                        fileElement.textContent = event.target.result;
                    } else {
                        fileElement = document.createElement('p');
                        fileElement.textContent = 'File format not supported for preview.';
                    }

                    // Menambahkan elemen file ke dalam grid
                    fileItem.appendChild(fileElement);
                    const fileNameElement = document.createElement('span');
                    fileNameElement.textContent = fileName;
                    fileItem.appendChild(fileNameElement);

                    // Menambahkan event listener untuk membuka file saat diklik
                    fileItem.addEventListener('click', () => openFileViewer(fileData, fileType));

                    fileGrid.appendChild(fileItem);
                };

                reader.readAsDataURL(file); // Mengonversi file menjadi data base64
            });
        }

        // Membuka file di viewer
        function openFileViewer(fileData, fileType) {
            let viewerElement;

            // Menyesuaikan elemen berdasarkan tipe file
            if (fileType.startsWith('image/')) {
                viewerElement = document.createElement('img');
                viewerElement.src = fileData;
            } else if (fileType.startsWith('audio/')) {
                viewerElement = document.createElement('audio');
                viewerElement.controls = true;
                viewerElement.src = fileData;
            } else if (fileType.startsWith('video/')) {
                viewerElement = document.createElement('video');
                viewerElement.controls = true;
                viewerElement.src = fileData;
            } else if (fileType === 'application/pdf') {
                viewerElement = document.createElement('iframe');
                viewerElement.src = fileData;
                viewerElement.width = "100%";
                viewerElement.height = "600px";
            } else if (fileType === 'text/plain') {
                viewerElement = document.createElement('pre');
                viewerElement.textContent = fileData;
            } else {
                viewerElement = document.createElement('p');
                viewerElement.textContent = 'File format not supported for preview.';
            }

            // Menampilkan file di jendela viewer
            fileContent.innerHTML = ''; // Bersihkan konten sebelumnya
            fileContent.appendChild(viewerElement);
            fileViewer.style.display = 'flex';
        }

        // Menutup viewer
        closeBtn.addEventListener('click', () => {
            fileViewer.style.display = 'none';
        });

        // Event listener untuk memilih file
        fileInput.addEventListener('change', (event) => {
            const files = event.target.files;
            saveAndDisplayFiles(files); // Menyimpan dan menampilkan file
        });
    </script>
</body>
</html>
