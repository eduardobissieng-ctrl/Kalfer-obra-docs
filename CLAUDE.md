# Documentos SST Kalfer

Aplicativo web da Kalfer Construção Civil para controlar os documentos de Segurança do Trabalho (SST) dos colaboradores das obras: cadastro de colaboradores, documentos com emissão/vencimento, anexos, alertas de vencimento, pendências e conferência de datas.

- **Site publicado:** https://eduardobissieng-ctrl.github.io/Kalfer-obra-docs/ (GitHub Pages, branch `main`, arquivo `index.html` na raiz)
- **Responsável:** Eduardo Bissi (Kalfer). Responda sempre em português, em linguagem simples (ele não é programador).

## Estrutura

- Tudo fica em um único arquivo: `index.html` (HTML + CSS + JavaScript, sem build, sem framework).
- A logo da Kalfer está embutida em base64 na constante `LOGO_DATA_URI` (uma linha enorme). Nunca leia nem reescreva essa linha; edite o arquivo com substituições pontuais.
- Bibliotecas por CDN, carregadas só quando necessário:
  - JSZip (cdnjs) para baixar anexos em .zip
  - pdf.js 3.11.174 (cdnjs) e Tesseract.js 5.1.1 (jsDelivr, idioma `por`) para a conferência de datas

## Banco de dados (Supabase)

- Projeto Supabase **Documentos-obra**: `https://cytygqcrzcdhhskwjgit.supabase.co`, acessado por REST com a chave *publishable* (pública, segura no front-end; a segurança vem do RLS).
- Login por e-mail e senha (Supabase Auth). Usuários são criados pelo Eduardo no painel do Supabase (Authentication > Users).
- Tabelas:
  - `employees` (id, name, role, `waived_docs text[]` = documentos marcados como "não se aplica")
  - `documents` (id, employee_id, type, issue_date, expiry_date, responsible_name, responsible_email, file_name, file_path, alert_sent_at)
  - `document_checks` (document_id text PK, result jsonb, checked_at) = resultado da conferência de datas dos anexos
  - `audit_log` = histórico de alterações (preenchido por trigger no banco)
- Storage: bucket privado `anexos` (acesso sempre pelo endpoint autenticado, nunca URL pública).
- Mudanças de banco não são feitas pelo app: entregue um script SQL idempotente (`if not exists`) e explique ao Eduardo, passo a passo, como rodar no **SQL Editor** do Supabase. Scripts já aplicados: `ativar-nao-se-aplica.sql` e `ativar-conferencia-datas.sql`.

## Regras de negócio

- **Documentos obrigatórios** (constante `REQUIRED_DOCS`): Checklist EPI, Foto SISCAER, Declaração de Prestação de Serviços, ASO, SISCAER, NR 06, NR 10, NR 12, NR 18, NR 35, Termo de Ciência, Ordem de Serviço, Contrato de Trabalho ou Ficha de Registro.
- O reconhecimento do tipo ignora acentos, maiúsculas e hífens (`normDocType`) e aceita variações:
  - "Ficha de EPI" conta como Checklist EPI
  - "OS" / "O.S." = Ordem de Serviço
  - "CTPS", "Carteira de Trabalho" e "FRE" = Contrato de Trabalho ou Ficha de Registro
  - "NR-35 (Altura)", "NR35", "NR 35" etc. = NR 35
- Não renomeie os rótulos de `REQUIRED_DOCS`: eles ficam gravados em `employees.waived_docs`.
- Painel: cartões Colaboradores, Atrasados, A vencer (30 dias), Em dia, Documentos pendentes, Datas divergentes nos anexos. Clicar num nome nos painéis abre o documento do colaborador.
- Conferência de datas: lê o anexo (texto do PDF ou OCR), procura as datas e compara com emissão/vencimento. Validade padrão quando o vencimento não está escrito: ASO 12 meses, NR 10 24, NR 35 24, NR 33 12 (constante `VALIDITY_RULES`, tolerância 7 dias).

## Ao publicar uma mudança

1. Aumente `APP_VERSION` (formato `AAAAMMDDNN`, ex.: `2026111601`). O app compara essa versão com a publicada e se atualiza sozinho nos navegadores da equipe; sem aumentar, ninguém recebe a versão nova.
2. Teste no navegador (Playwright com dados simulados do Supabase) antes de entregar: sem erros no console, telas principais funcionando.
3. O GitHub Pages publica a partir da branch `main`. Quando o trabalho for numa branch, abra o Pull Request e explique ao Eduardo que é só clicar em **Merge** para o site atualizar (1 a 2 minutos).

## Outro projeto relacionado

O **Gerador de APR** (Análise Preliminar de Risco) ainda é uma página separada no claude.ai, fora deste repositório.
