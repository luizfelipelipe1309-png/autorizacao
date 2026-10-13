<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gerador de Autorização - IFCE</title>
    <!-- html2pdf library for PDF generation -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    <style>
        :root {
            --primary-color: #2e7d32; /* Verde IFCE */
            --bg-color: #f5f5f5;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 700px;
            margin: 0 auto;
            background: #ffffff;
            padding: 30px;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }
        h2 {
            text-align: center;
            color: #333;
            margin-bottom: 25px;
        }
        .form-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
        }
        .form-group {
            display: flex;
            flex-direction: column;
        }
        .form-group.full-width {
            grid-column: span 2;
        }
        label {
            font-weight: 600;
            margin-bottom: 5px;
            font-size: 14px;
            color: #555;
        }
        input[type="text"], input[type="date"], input[type="number"] {
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 4px;
            font-size: 14px;
            transition: border-color 0.3s;
        }
        input:focus {
            border-color: var(--primary-color);
            outline: none;
        }
        .btn-generate {
            display: block;
            width: 100%;
            background-color: var(--primary-color);
            color: white;
            border: none;
            padding: 15px;
            font-size: 16px;
            font-weight: bold;
            border-radius: 4px;
            margin-top: 25px;
            cursor: pointer;
            transition: background-color 0.3s;
        }
        .btn-generate:hover {
            background-color: #1b5e20;
        }

        /* ---------------------------------------------------
           Estilos específicos para o documento PDF (A4)
           Fica oculto na tela e só aparece na hora de gerar
        --------------------------------------------------- */
        #pdf-container {
            position: absolute;
            left: -9999px;
            top: -9999px;
        }
        #pdf-content {
            display: none;
            width: 210mm;
            padding: 25mm 20mm; /* Margens padrão A4 */
            background: white;
            box-sizing: border-box;
            font-family: "Times New Roman", Times, serif;
            font-size: 12pt;
            line-height: 1.5;
            color: #000;
        }
        .pdf-header {
            text-align: center;
            font-weight: bold;
            font-size: 11pt;
            margin-bottom: 40px;
            line-height: 1.2;
        }
        .pdf-header img {
            width: 80px;
            height: auto;
            margin-bottom: 10px;
        }
        .pdf-title {
            text-align: center;
            font-weight: bold;
            text-decoration: underline;
            font-size: 12pt;
            margin-bottom: 30px;
        }
        .pdf-body {
            text-align: justify;
            margin-bottom: 50px;
        }
        .pdf-date {
            text-align: right;
            margin-bottom: 50px;
        }
        .pdf-footer {
            font-size: 10pt;
            margin-bottom: 40px;
        }
        .pdf-signatures {
            display: flex;
            justify-content: space-between;
            text-align: center;
            font-size: 11pt;
            margin-top: 50px;
        }
        .pdf-signature-box {
            width: 45%;
            border-top: 1px solid #000;
            padding-top: 5px;
        }
        strong.placeholder {
            font-weight: normal;
        }
    </style>
</head>
<body>

    <div class="container">
        <h2>Preencher Autorização</h2>
        <div class="form-grid">
            <div class="form-group full-width">
                <label>Nome do Responsável Legal:</label>
                <input type="text" id="respNome" placeholder="Ex: Maria da Silva">
            </div>
            <div class="form-group">
                <label>RG do Responsável:</label>
                <input type="text" id="respRg" placeholder="Ex: 2008123456-7">
            </div>
            <div class="form-group">
                <label>Órgão Emissor (Resp.):</label>
                <input type="text" id="respOrgao" placeholder="Ex: SSP/CE">
            </div>
            <div class="form-group">
                <label>Data de Expedição do RG:</label>
                <input type="date" id="respDataExp">
            </div>
            <div class="form-group">
                <label>Dia da Assinatura (Out/2026):</label>
                <input type="number" id="diaAssinatura" min="1" max="31" placeholder="Ex: 20">
            </div>

            <div class="form-group full-width" style="margin-top: 10px; border-top: 1px solid #eee; padding-top: 15px;">
                <label>Nome Completo do Aluno:</label>
                <input type="text" id="alunoNome" placeholder="Ex: João da Silva">
            </div>
            <div class="form-group">
                <label>RG do Aluno:</label>
                <input type="text" id="alunoRg" placeholder="Ex: 2020123456-8">
            </div>
            <div class="form-group">
                <label>CPF do Aluno:</label>
                <input type="text" id="alunoCpf" placeholder="Ex: 123.456.789-00">
            </div>
        </div>

        <button class="btn-generate" onclick="gerarPdf()">Gerar Autorização em PDF</button>
    </div>

    <!-- Template do PDF Oculto -->
    <div id="pdf-container">
        <div id="pdf-content">
            <div class="pdf-header">
                <img src="data:image/png;base64,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" alt="Brasão da República"><br>
                MINISTÉRIO DA EDUCAÇÃO<br>
                SECRETARIA DE EDUCAÇÃO PROFISSIONAL E TECNOLÓGICA<br>
                INSTITUTO FEDERAL DE EDUCAÇÃO, CIÊNCIA E TECNOLOGIA DO CEARÁ – CAMPUS JUAZEIRO DO NORTE
            </div>
            
            <div class="pdf-title">
                FICHA DE AUTORIZAÇÃO DE MENOR – VISITA TÉCNICA
            </div>

            <div class="pdf-body">
                Eu, <strong class="placeholder" id="outRespNome">______________________</strong>, portador(a) do RG nº <strong class="placeholder" id="outRespRg">___________</strong>, órgão emissor <strong class="placeholder" id="outRespOrgao">_______</strong>, com data de expedição em <strong class="placeholder" id="outRespDataExp">__/__/____</strong>, autorizo o(a) menor <strong class="placeholder" id="outAlunoNome">______________________</strong>, portador(a) do RG nº <strong class="placeholder" id="outAlunoRg">___________</strong> e CPF de nº <strong class="placeholder" id="outAlunoCpf">___________</strong>, a participar das visitas técnicas relacionadas às disciplinas de PRÁTICA PROFISSIONAL do CURSO DE EDIFICAÇÕES (3º ANO), localizado na cidade do Fortaleza, entre os dias 25/11/2026 (6h) e 28/11/2026 (22h). A visita técnica ocorrerá sob a responsabilidade dos professores <b>Maria Regilene Gonçalves de Alcantara</b>, <b>Antonio Alex Matias Lêu</b> e <b>Thiago Ribeiro Francelino</b> todos professores e servidor do IFCE campus Juazeiro do Norte.
            </div>

            <div class="pdf-date">
                Juazeiro do Norte, <span id="outDiaAssinatura">__</span> de outubro de 2026.
            </div>

            <div class="pdf-footer">
                * Esta autorização deverá permanecer junto aos documentos do(a) menor, devidamente assinada.
            </div>

            <div class="pdf-signatures">
                <div class="pdf-signature-box">
                    (Assinatura do responsável legal do menor)
                </div>
                <div class="pdf-signature-box">
                    (Assinatura do servidor responsável)
                </div>
            </div>
        </div>
    </div>

    <script>
        function formatarData(dataISO) {
            if (!dataISO) return '__/__/____';
            const partes = dataISO.split('-');
            return `${partes[2]}/${partes[1]}/${partes[0]}`;
        }

        function gerarPdf() {
            document.getElementById('outRespNome').innerText = document.getElementById('respNome').value || '______________________';
            document.getElementById('outRespRg').innerText = document.getElementById('respRg').value || '___________';
            document.getElementById('outRespOrgao').innerText = document.getElementById('respOrgao').value || '_______';
            document.getElementById('outRespDataExp').innerText = formatarData(document.getElementById('respDataExp').value);
            
            document.getElementById('outAlunoNome').innerText = document.getElementById('alunoNome').value || '______________________';
            document.getElementById('outAlunoRg').innerText = document.getElementById('alunoRg').value || '___________';
            document.getElementById('outAlunoCpf').innerText = document.getElementById('alunoCpf').value || '___________';
            document.getElementById('outDiaAssinatura').innerText = document.getElementById('diaAssinatura').value || '__';

            const elemento = document.getElementById('pdf-content');
            elemento.style.display = 'block';

            const opt = {
                margin:       0,
                filename:     `Autorizacao_${document.getElementById('alunoNome').value || 'Aluno'}.pdf`.replace(/\s+/g, '_'),
                image:        { type: 'jpeg', quality: 0.98 },
                html2canvas:  { scale: 2, useCORS: true },
                jsPDF:        { unit: 'mm', format: 'a4', orientation: 'portrait' }
            };

            html2pdf().set(opt).from(elemento).save().then(() => {
                elemento.style.display = 'none';
            });
        }
    </script>
</body>
</html>
