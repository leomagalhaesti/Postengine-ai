# ⚡ PostEngineAI — Autonomous LinkedIn Content & Media Pipeline

[![n8n](https://img.shields.io/badge/n8n-Workflow-FF6D5A?style=flat-square&logo=n8n)](https://n8n.io/)
[![Gemini](https://img.shields.io/badge/Google-Gemini_Flash-4285F4?style=flat-square&logo=googlegemini)](https://ai.google.dev/)
[![Pixabay](https://img.shields.io/badge/Pixabay-API-02B875?style=flat-square&logo=pixabay)](https://pixabay.com/)
[![Notion](https://img.shields.io/badge/Notion-Database-000000?style=flat-square&logo=notion)](https://notion.so/)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-API-0A66C2?style=flat-square&logo=linkedin)](https://linkedin.com/)

---

## 🇧🇷 PORTUGUÊS

Uma arquitetura de automacao end-to-end desenvolvida no n8n para criacao, validacao, busca de midia e publicacao autonoma de conteudos no LinkedIn duas vezes por dia, integrando Inteligencia Artificial (Google Gemini), busca automatica de imagens (Pixabay API) e registro de historico via Notion.

📌 Visao Geral do Projeto
Este projeto resolve o desafio de manter consistencia de publicacao no LinkedIn sem a necessidade de alimentacao manual diaria.

A partir de um agendamento (Schedule Trigger, 09h e 22h), o workflow consulta no Notion os temas ja publicados, sorteia area, formato e tom do post, gera o texto com o Gemini, valida e limpa a resposta, busca fotos do tema na API do Pixabay, escolhe uma foto que ainda nao foi usada, realiza o download binario do arquivo em memoria e publica o post unificado diretamente no feed do LinkedIn. Por fim, grava o historico interno e registra o post no Notion.

📐 Arquitetura da Solucao

[ Schedule Trigger ] ──► (Execucao as 09h e 22h / Cron 24/7 na VPS)
         │
         ▼
[ Notion Database ] ──► (Le os temas ja publicados para evitar repeticao)
         │
         ▼
[ Code: tema do dia ] ──► (Sorteia area, formato, tom e gancho; manha = dicas, noite = reflexoes)
         │
         ▼
[ Google Gemini Flash ] ──► (Engenharia de prompt com saida estruturada TITULO / BUSCA / POST)
         │
         ▼
[ Code: validacao ] ──► (Remove markdown e caracteres especiais, valida tamanho e titulo)
         │
         ▼
[ Pixabay REST API ] ──► (Busca do tema + busca de reserva, 50 fotos cada)
         │
         ▼
[ Code: escolha da foto ] ──► (Descarta fotos ja usadas pelo ID)
         │
         ▼
[ HTTP Binary Download ] ──► (Converte URL do CDN em buffer binario em memoria)
         │
         ▼
[ LinkedIn API v2 ] ──► (Publicacao unificada: texto + imagem + credito do fotografo)
         │
         ▼
[ Historico + Notion ] ──► (Grava fotos e temas usados e registra o post como "Concluido")

🛠️ Desafios Tecnicos & Solucoes de Engenharia

• Processamento de Midia Binaria (Buffer Stream): A API v2 do LinkedIn rejeita URLs externas diretas de imagem. A solucao utiliza uma camada HTTP intermediaria que faz o download dos bytes reais do CDN do Pixabay para a memoria temporaria do n8n, transmitindo os dados via multipart/form-data.

• Saida Estruturada e Validacao: O LLM responde em um formato fixo (TITULO / BUSCA / POST). Um no de codigo separa os campos, remove markdown e caracteres que quebram a publicacao, corta textos longos e interrompe a execucao se o post vier vazio ou sem titulo.

• Algoritmo Anti-Duplicacao: Os IDs das ultimas 300 fotos publicadas ficam no static data do workflow e sao descartados na escolha; os temas ja publicados (Notion + memoria interna) vao no prompt, e as ultimas 3 areas nao sao sorteadas de novo. O historico so e gravado em execucoes agendadas, nao em testes manuais.

⚙️ Configuracao

1. Importe o arquivo `PostEngineAI.json` no n8n.
2. Crie as credenciais e selecione cada uma nos nos correspondentes:
   • Gemini: tipo Header Auth, Name `x-goog-api-key`, Value = chave do Google AI Studio.
   • Pixabay: tipo Query Auth, Name `key`, Value = chave da API do Pixabay.
   • Notion: integracao com acesso a base de posts.
   • LinkedIn: OAuth2 com "Organization Support" e "Legacy" desligados; o app precisa dos produtos "Share on LinkedIn" e "Sign In with LinkedIn using OpenID Connect".
3. Ajuste a base do Notion e a pessoa do LinkedIn nos nos, e o fuso horario da instancia (`GENERIC_TIMEZONE`).
4. Ative o workflow.

Nenhuma chave fica no JSON: o arquivo guarda apenas o nome e o ID das credenciais.

---

## 🇺🇸 ENGLISH

An end-to-end automation architecture built with n8n for autonomous creation, validation, media retrieval, and twice-daily publishing of content to LinkedIn, integrating Artificial Intelligence (Google Gemini), automated image retrieval (Pixabay API), and history tracking via Notion.

📌 Project Overview
This project solves the challenge of maintaining consistent publication activity on LinkedIn without requiring daily manual input.

Triggered by a schedule (Schedule Trigger, 9 AM and 10 PM), the workflow reads previously published topics from Notion, draws the post's area, format and tone, generates the copy with Gemini, validates and cleans the response, fetches matching photos from the Pixabay API, picks one that has not been used before, downloads the raw binary file into memory, and publishes the unified post directly to the LinkedIn feed. Finally, it stores its internal history and logs the post in Notion.

📐 Solution Architecture

[ Schedule Trigger ] ──► (Runs at 9 AM and 10 PM / 24/7 Cron on VPS)
         │
         ▼
[ Notion Database ] ──► (Reads published topics to avoid repetition)
         │
         ▼
[ Code: daily topic ] ──► (Draws area, format, tone and hook; morning = tips, evening = reflections)
         │
         ▼
[ Google Gemini Flash ] ──► (Prompt engineering with structured TITULO / BUSCA / POST output)
         │
         ▼
[ Code: validation ] ──► (Strips markdown and special characters, validates length and title)
         │
         ▼
[ Pixabay REST API ] ──► (Topic search + fallback search, 50 photos each)
         │
         ▼
[ Code: photo selection ] ──► (Discards already used photos by ID)
         │
         ▼
[ HTTP Binary Download ] ──► (Converts CDN URL into in-memory binary buffer)
         │
         ▼
[ LinkedIn API v2 ] ──► (Unified publishing: text + image + photographer credit)
         │
         ▼
[ History + Notion ] ──► (Stores used photos and topics and logs the post as "Done")

🛠️ Technical Challenges & Engineering Solutions

• Binary Payload Processing (Buffer Stream): LinkedIn API v2 rejects direct external image URLs. The solution implements an intermediate HTTP download layer that streams raw bytes from Pixabay's CDN to n8n's temporary memory, transmitting data via multipart/form-data.

• Structured Output and Validation: The LLM answers in a fixed format (TITULO / BUSCA / POST). A code node splits the fields, strips markdown and characters that break publishing, trims long texts and stops the execution if the post comes back empty or without a title.

• Anti-Duplication Algorithm: The IDs of the last 300 published photos are kept in the workflow static data and discarded during selection; previously published topics (Notion + internal memory) are sent in the prompt, and the last 3 areas are not drawn again. History is only stored on scheduled executions, not on manual tests.

⚙️ Setup

1. Import `PostEngineAI.json` into n8n.
2. Create the credentials and select each one in its nodes:
   • Gemini: Header Auth type, Name `x-goog-api-key`, Value = Google AI Studio key.
   • Pixabay: Query Auth type, Name `key`, Value = Pixabay API key.
   • Notion: integration with access to the posts database.
   • LinkedIn: OAuth2 with "Organization Support" and "Legacy" turned off; the app needs the "Share on LinkedIn" and "Sign In with LinkedIn using OpenID Connect" products.
3. Adjust the Notion database and the LinkedIn person in the nodes, and the instance timezone (`GENERIC_TIMEZONE`).
4. Activate the workflow.

No keys are stored in the JSON: the file only holds credential names and IDs.

---

## 🧰 Tech Stack

• Orchestrator / Orquestrador: n8n Self-Hosted (Hostinger Linux VPS)
• AI Engine / Motor de IA: Google Gemini API (Gemini 3.5 Flash)
• Media Provider / Midias: Pixabay REST API
• CMS & Database: Notion API
• Target Platform / Destino: LinkedIn API v2

---

## 👨‍💻 Autor / Author

Leonardo Magalhaes
Desenvolvedor & Entusiasta em Automacao de Processos | Process Automation & Software Engineering
GitHub: [https://github.com/leomagalhaesti](https://github.com/leomagalhaesti)
