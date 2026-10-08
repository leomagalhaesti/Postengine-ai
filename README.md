# ⚡ PostEngineAI — Autonomous LinkedIn Content & Media Pipeline

[![n8n](https://img.shields.io/badge/n8n-Workflow-FF6D5A?style=flat-square&logo=n8n)](https://n8n.io/)
[![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o--mini-412991?style=flat-square&logo=openai)](https://openai.com/)
[![Pixabay](https://img.shields.io/badge/Pixabay-API-02B875?style=flat-square&logo=pixabay)](https://pixabay.com/)
[![Notion](https://img.shields.io/badge/Notion-Database-000000?style=flat-square&logo=notion)](https://notion.so/)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-API-0A66C2?style=flat-square&logo=linkedin)](https://linkedin.com/)

---

## 🇧🇷 PORTUGUÊS

Uma arquitetura de automacao end-to-end desenvolvida no n8n para criacao, formatacao, busca de midia e publicacao autonoma de conteudos diarios no LinkedIn, integrando Inteligencia Artificial (OpenAI/ChatGPT), busca automatica de imagens (Pixabay API) e gestao de fluxo via Notion.

📌 Visao Geral do Projeto
Este projeto resolve o desafio de manter consistencia de publicacao no LinkedIn sem a necessidade de alimentacao manual diaria.

A partir de um agendamento cronometrado (Schedule Trigger), o workflow consulta um banco de dados no Notion, gera um texto estrategico com a OpenAI (formatado com caracteres Unicode em negrito/italico e emojis), busca uma imagem de alta resolucao correspondente ao tema na API do Pixabay, realiza o download binario do arquivo em memoria e publica o post unificado diretamente no feed do LinkedIn.

📐 Arquitetura da Solucao

[ Schedule Trigger ] ──► (Execucao diaria programada / Cron 24/7 na VPS)
         │
         ▼
[ Notion Database ] ──► (Consulta topicos e pautas agendadas do dia)
         │
         ▼
[ OpenAI GPT-4o-mini ] ──► (Engenharia de prompt + conversao Unicode Bold/Italic)
         │
         ▼
[ Pixabay REST API ] ──► (Sorteio randomico de fotos em alta resolucao por tema)
         │
         ▼
[ HTTP Binary Download ] ──► (Converte URL do CDN em buffer binario em memoria)
         │
         ▼
[ LinkedIn API v2 ] ──► (Publicacao unificada: Legenda formatada + Anexo de imagem)
         │
         ▼
[ Notion Update ] ──► (Atualiza o status para "Concluido" e registra log)

🛠️ Desafios Tecnicos & Solucoes de Engenharia

• Processamento de Midia Binaria (Buffer Stream): A API v2 do LinkedIn rejeita URLs externas diretas de imagem. A solucao utiliza uma camada HTTP intermediaria que faz o download dos bytes reais do CDN do Pixabay para a memoria temporaria do n8n, transmitindo os dados via multipart/form-data.

• Formatacao Visual em Unicode: Como as APIs de redes sociais nao interpretam tags Markdown nativas (como **texto**), o pipeline orienta o LLM a converter palavras-chave e titulos em caracteres Unicode Mathematical Bold/Italic, garantindo destaque no feed sem quebrar o envio.

• Algoritmo Anti-Duplicacao: Sorteio randomico de paginas e indices nos resultados da API do Pixabay e prompts dinamicos que exigem introducoes ineditas a cada execucao.

---

## 🇺🇸 ENGLISH

An end-to-end automation architecture built with n8n for autonomous creation, formatting, media retrieval, and daily publishing of content to LinkedIn, integrating Artificial Intelligence (OpenAI/ChatGPT), automated image retrieval (Pixabay API), and workflow management via Notion.

📌 Project Overview
This project solves the challenge of maintaining consistent publication activity on LinkedIn without requiring daily manual input.

Triggered by a scheduled time event (Schedule Trigger), the workflow queries a database in Notion, generates strategic copy using OpenAI (formatted with bold/italic Unicode characters and emojis), fetches a high-resolution image matching the topic from the Pixabay API, downloads the raw binary file into memory, and publishes the unified post directly to the LinkedIn feed.

📐 Solution Architecture

[ Schedule Trigger ] ──► (Daily scheduled execution / 24/7 Cron on VPS)
         │
         ▼
[ Notion Database ] ──► (Queries today's scheduled topics and tasks)
         │
         ▼
[ OpenAI GPT-4o-mini ] ──► (Prompt engineering + Unicode Bold/Italic conversion)
         │
         ▼
[ Pixabay REST API ] ──► (Randomized selection of high-res photos by topic)
         │
         ▼
[ HTTP Binary Download ] ──► (Converts CDN URL into in-memory binary buffer)
         │
         ▼
[ LinkedIn API v2 ] ──► (Unified publishing: Formatted caption + Image attachment)
         │
         ▼
[ Notion Update ] ──► (Updates status to "Done" and logs output)

🛠️ Technical Challenges & Engineering Solutions

• Binary Payload Processing (Buffer Stream): LinkedIn API v2 rejects direct external image URLs. The solution implements an intermediate HTTP download layer that streams raw bytes from Pixabay's CDN to n8n's temporary memory, transmitting data via multipart/form-data.

• Dynamic Unicode Formatting Engine: Social feed APIs do not render standard Markdown tags natively (**bold**). The pipeline instructs the LLM to output Unicode Mathematical Bold/Italic characters, ensuring proper visual emphasis without formatting breaks.

• Anti-Duplication Algorithm: Randomized page and index fetching from Pixabay API results alongside dynamic prompts enforcing unique hooks for every execution.

---

## 🧰 Tech Stack

• Orchestrator / Orquestrador: n8n Self-Hosted (Hostinger Linux VPS)
• AI Engine / Motor de IA: OpenAI API (GPT-4o-mini)
• Media Provider / Midias: Pixabay REST API
• CMS & Database: Notion API
• Target Platform / Destino: LinkedIn API v2

---

## 👨‍💻 Autor / Author

Leonardo Magalhaes
Desenvolvedor & Entusiasta em Automacao de Processos | Process Automation & Software Engineering
GitHub: [https://github.com/leomagalhaesti](https://github.com/leomagalhaesti)
