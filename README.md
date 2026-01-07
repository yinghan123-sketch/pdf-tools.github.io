<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PDF加页码工具</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf-lib/1.17.1/pdf-lib.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }

        .container {
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
            padding: 40px;
            max-width: 600px;
            width: 100%;
        }

        h1 {
            color: #333;
            text-align: center;
            margin-bottom: 30px;
            font-size: 2rem;
        }

        .upload-area {
            border: 2px dashed #667eea;
            border-radius: 10px;
            padding: 40px;
            text-align: center;
            margin-bottom: 30px;
            transition: all 0.3s ease;
            cursor: pointer;
        }

        .upload-area:hover {
            border-color: #764ba2;
            background-color: #f8f9ff;
        }

        .upload-area.dragover {
            background-color: #e8ebff;
            border-color: #764ba2;
        }

        .upload-icon {
            font-size: 48px;
            color: #667eea;
            margin-bottom: 10px;
        }

        input[type="file"] {
            display: none;
        }

        .options {
            margin-bottom: 30px;
        }

        .option-group {
            margin-bottom: 20px;
        }

        label {
            display: block;
            margin-bottom: 8px;
            color: #555;
            font-weight: 500;
        }

        input[type="text"],
        input[type="number"],
        select {
            width: 100%;
            padding: 10px 15px;
            border: 1px solid #ddd;
            border-radius: 8px;
            font-size: 16px;
            transition: border-color 0.3s ease;
        }

        input[type="text"]:focus,
        input[type="number"]:focus,
        select:focus {
            outline: none;
            border-color: #667eea;
        }

        .font-options {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
        }

        .color-picker-wrapper {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        input[type="color"] {
            width: 50px;
            height: 40px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
        }

        .position-options {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
        }

        .position-btn {
            padding: 10px;
            border: 2px solid #ddd;
            border-radius: 8px;
            background: white;
            cursor: pointer;
            transition: all 0.3s ease;
            text-align: center;
        }

        .position-btn:hover {
            border-color: #667eea;
        }

        .position-btn.active {
            background: #667eea;
            color: white;
            border-color: #667eea;
        }

        .process-btn {
            width: 100%;
            padding: 15px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            border-radius: 10px;
            font-size: 18px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .process-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 20px rgba(102, 126, 234, 0.3);
        }

        .process-btn:disabled {
            background: #ccc;
            cursor: not-allowed;
            transform: none;
            box-shadow: none;
        }

        .loading {
            display: none;
            text-align: center;
            margin-top: 20px;
        }

        .loading-spinner {
            border: 3px solid #f3f3f3;
            border-top: 3px solid #667eea;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
            margin: 0 auto 10px;
        }

        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .result {
            display: none;
            text-align: center;
            margin-top: 20px;
            padding: 20px;
            background: #f8f9ff;
            border-radius: 10px;
        }

        .download-btn {
            display: inline-block;
            padding: 12px 30px;
            background: #4caf50;
            color: white;
            text-decoration: none;
            border-radius: 8px;
            margin-top: 10px;
            transition: all 0.3s ease;
        }

        .download-btn:hover {
            background: #45a049;
            transform: translateY(-2px);
        }

        .preview {
            margin-top: 20px;
            text-align: center;
        }

        .preview canvas {
            max-width: 100%;
            border: 1px solid #ddd;
            border-radius: 8px;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>📄 PDF加页码工具</h1>
        
        <div class="upload-area" id="uploadArea">
            <div class="upload-icon">📎</div>
            <p>点击上传PDF文件或拖拽文件到此处</p>
            <input type="file" id="fileInput" accept=".pdf">
        </div>

        <div class="options" id="options" style="display: none;">
            <div class="option-group">
                <label for="pageNumberFormat">页码格式</label>
                <select id="pageNumberFormat">
                    <option value="1, 2, 3...">1, 2, 3...</option>
                    <option value="第1页, 第2页, 第3页...">第1页, 第2页, 第3页...</option>
                    <option value="Page 1, Page 2, Page 3...">Page 1, Page 2, Page 3...</option>
                    <option value="- 1 -, - 2 -, - 3 -...">- 1 -, - 2 -, - 3 -...</option>
                </select>
            </div>

            <div class="font-options">
                <div class="option-group">
                    <label for="fontSize">字体大小</label>
                    <input type="number" id="fontSize" value="12" min="8" max="24">
                </div>
                <div class="option-group">
                    <label for="fontFamily">字体</label>
                    <select id="fontFamily">
                        <option value="Helvetica">Helvetica</option>
                        <option value="Times-Roman">Times New Roman</option>
                        <option value="Courier">Courier</option>
                    </select>
                </div>
            </div>

            <div class="option-group">
                <label>字体颜色</label>
                <div class="color-picker-wrapper">
                    <input type="color" id="fontColor" value="#000000">
                    <span>选择颜色</span>
                </div>
            </div>

            <div class="option-group">
                <label>页码位置</label>
                <div class="position-options">
                    <button class="position-btn" data-position="bottom-center">底部居中</button>
                    <button class="position-btn" data-position="bottom-left">底部左侧</button>
                    <button class="position-btn" data-position="bottom-right">底部右侧</button>
                    <button class="position-btn" data-position="top-center">顶部居中</button>
                    <button class="position-btn" data-position="top-left">顶部左侧</button>
                    <button class="position-btn" data-position="top-right">顶部右侧</button>
                </div>
            </div>

            <div class="option-group">
                <label for="startPage">起始页码</label>
                <input type="number" id="startPage" value="1" min="1">
            </div>

            <div class="option-group">
                <label for="startFromPage">从第几页开始添加</label>
                <input type="number" id="startFromPage" value="1" min="1">
            </div>

            <button class="process-btn" id="processBtn">开始添加页码</button>
        </div>

        <div class="loading" id="loading">
            <div class="loading-spinner"></div>
            <p>正在处理PDF文件...</p>
        </div>

        <div class="result" id="result">
            <p>✅ PDF文件处理完成！</p>
            <a href="#" class="download-btn" id="downloadBtn">下载带页码的PDF</a>
        </div>

        <div class="preview" id="preview"></div>
    </div>

    <script>
        let pdfDoc = null;
        let selectedFile = null;
        let pagePosition = 'bottom-center';

        // 文件上传处理
        const uploadArea = document.getElementById('uploadArea');
        const fileInput = document.getElementById('fileInput');
        const options = document.getElementById('options');

        uploadArea.addEventListener('click', () => fileInput.click());

        fileInput.addEventListener('change', (e) => {
            handleFile(e.target.files[0]);
        });

        // 拖拽上传
        uploadArea.addEventListener('dragover', (e) => {
            e.preventDefault();
            uploadArea.classList.add('dragover');
        });

        uploadArea.addEventListener('dragleave', () => {
            uploadArea.classList.remove('dragover');
        });

        uploadArea.addEventListener('drop', (e) => {
            e.preventDefault();
            uploadArea.classList.remove('dragover');
            handleFile(e.dataTransfer.files[0]);
        });

        // 位置选择
        document.querySelectorAll('.position-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                document.querySelectorAll('.position-btn').forEach(b => b.classList.remove('active'));
                btn.classList.add('active');
                pagePosition = btn.dataset.position;
            });
        });

        // 默认选中底部居中
        document.querySelector('[data-position="bottom-center"]').classList.add('active');

        // 处理文件
        async function handleFile(file) {
            if (!file || file.type !== 'application/pdf') {
                alert('请选择PDF文件！');
                return;
            }

            selectedFile = file;
            uploadArea.innerHTML = `<p>已选择文件: ${file.name}</p>`;
            options.style.display = 'block';

            // 预览第一页
            const arrayBuffer = await file.arrayBuffer();
            pdfDoc = await PDFLib.PDFDocument.load(arrayBuffer);
            await previewFirstPage();
        }

        // 预览第一页
        async function previewFirstPage() {
            const preview = document.getElementById('preview');
            const pages = pdfDoc.getPages();
            if (pages.length > 0) {
                const { width, height } = pages[0].getSize();
                const canvas = document.createElement('canvas');
                canvas.width = 200;
                canvas.height = (height / width) * 200;
                canvas.style.border = '1px solid #ddd';
                canvas.style.marginTop = '10px';
                preview.innerHTML = '<p>PDF预览（第一页）</p>';
                preview.appendChild(canvas);
            }
        }

        // 处理PDF
        document.getElementById('processBtn').addEventListener('click', async () => {
            if (!pdfDoc) {
                alert('请先上传PDF文件！');
                return;
            }

            const loading = document.getElementById('loading');
            const result = document.getElementById('result');
            
            loading.style.display = 'block';
            result.style.display = 'none';

            try {
                const format = document.getElementById('pageNumberFormat').value;
                const fontSize = parseInt(document.getElementById('fontSize').value);
                const fontFamily = document.getElementById('fontFamily').value;
                const fontColor = document.getElementById('fontColor').value;
                const startPage = parseInt(document.getElementById('startPage').value);
                const startFromPage = parseInt(document.getElementById('startFromPage').value) - 1;

                const newPdfDoc = await PDFLib.PDFDocument.create();
                const pages = pdfDoc.getPages();

                for (let i = 0; i < pages.length; i++) {
                    const [copiedPage] = await newPdfDoc.copyPages(pdfDoc, [i]);
                    newPdfDoc.addPage(copiedPage);

                    if (i >= startFromPage) {
                        const currentPage = newPdfDoc.getPages()[i];
                        const { width, height } = currentPage.getSize();
                        
                        let pageNumber = i + startPage - startFromPage;
                        let pageText = '';
                        
                        switch(format) {
                            case '1, 2, 3...':
                                pageText = pageNumber.toString();
                                break;
                            case '第1页, 第2页, 第3页...':
                                pageText = `第${pageNumber}页`;
                                break;
                            case 'Page 1, Page 2, Page 3...':
                                pageText = `Page ${pageNumber}`;
                                break;
                            case '- 1 -, - 2 -, - 3 -...':
                                pageText = `- ${pageNumber} -`;
                                break;
                        }

                        // 计算位置
                        let x, y;
                        const margin = 50;
                        
                        switch(pagePosition) {
                            case 'bottom-center':
                                x = width / 2;
                                y = margin;
                                break;
                            case 'bottom-left':
                                x = margin;
                                y = margin;
                                break;
                            case 'bottom-right':
                                x = width - margin;
                                y = margin;
                                break;
                            case 'top-center':
                                x = width / 2;
                                y = height - margin;
                                break;
                            case 'top-left':
                                x = margin;
                                y = height - margin;
                                break;
                            case 'top-right':
                                x = width - margin;
                                y = height - margin;
                                break;
                        }

                        // 添加页码
                        currentPage.drawText(pageText, {
                            x: x,
                            y: y,
                            size: fontSize,
                            font: await newPdfDoc.embedFont(fontFamily),
                            color: hexToRgb(fontColor),
                            textAnchor: pagePosition.includes('center') ? 'middle' : 
                                       pagePosition.includes('left') ? 'start' : 'end',
                        });
                    }
                }

                // 生成新的PDF
                const pdfBytes = await newPdfDoc.save();
                const blob = new Blob([pdfBytes], { type: 'application/pdf' });
                const url = URL.createObjectURL(blob);

                document.getElementById('downloadBtn').href = url;
                document.getElementById('downloadBtn').download = `带页码_${selectedFile.name}`;

                loading.style.display = 'none';
                result.style.display = 'block';

            } catch (error) {
                console.error('处理PDF时出错:', error);
                alert('处理PDF时出错，请重试！');
                loading.style.display = 'none';
            }
        });

        // 颜色转换函数
        function hexToRgb(hex) {
            const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
            return result ? {
                r: parseInt(result[1], 16) / 255,
                g: parseInt(result[2], 16) / 255,
                b: parseInt(result[3], 16) / 255,
            } : { r: 0, g: 0, b: 0 };
        }
    </script>
</body>
</html>
