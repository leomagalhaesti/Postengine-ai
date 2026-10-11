# ⚡ PostEngineAI — Autonomous LinkedIn Content & Media Pipeline

[![n8n](https://img.shields.io/badge/n8n-Workflow-FF6D5A?style=flat-square&logo=n8n)](https://n8n.io/)
[![Gemini](https://img.shields.io/badge/Google-Gemini_Flash-4285F4?style=flat-square&logo=googlegemini)](https://ai.google.dev/)
[![Pixabay](https://img.shields.io/badge/Pixabay-API-02B875?style=flat-square&logo=pixabay)](https://pixabay.com/)
[![Notion](https://img.shields.io/badge/Notion-Database-000000?style=flat-square&logo=notion)](https://notion.so/)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-API-0A66C2?style=flat-square&logo=linkedin)](https://linkedin.com/)
[![GitHub](https://img.shields.io/badge/GitHub-API-181717?style=flat-square&logo=github)](https://docs.github.com/rest)

---

## 🇧🇷 PORTUGUÊS

Uma arquitetura de automacao end-to-end desenvolvida no n8n para criacao, validacao, geracao de midia e publicacao autonoma de conteudos diarios no LinkedIn, integrando Inteligencia Artificial (Google Gemini), busca automatica de imagens (Pixabay API), leitura de projetos no GitHub, geracao de carrossel em PDF e registro de historico via Notion.

📌 Visao Geral do Projeto
Este projeto resolve o desafio de manter consistencia de publicacao no LinkedIn sem a necessidade de alimentacao manual diaria, e sem que todos os posts saiam no mesmo molde.

O workflow dispara todos os dias as 08h e espera um intervalo sorteado, de modo que o post sai em um horario diferente a cada dia, entre 08h10 e 10h40. Em seguida ele le os repositorios publicos do autor no GitHub e os temas ja publicados no Notion, e sorteia o tipo do post, a area, o formato, o tom e o tamanho do texto. O Gemini escreve o conteudo em formato estruturado, um no de codigo valida e limpa a resposta, e o post segue pelo caminho do tipo sorteado. Por fim, o workflow grava o historico interno e registra o post no Notion.

🎲 Tipos de Post

| Tipo | Peso no sorteio | O que e publicado |
|---|---|---|
| Foto | 35% | Texto com uma foto do Pixabay que ainda nao foi usada, com credito do fotografo |
| So texto | 22% | Texto sem imagem, em geral mais curto |
| Bastidor | 18% | Texto sobre um projeto do autor, escrito a partir do README do repositorio, com link para o codigo |
| Carrossel | 17% | Legenda curta e um PDF de 5 a 7 slides gerado dentro do proprio n8n |
| Enquete | 8% | Texto curto e uma enquete de 3 ou 4 opcoes, aberta por 7 dias |

O mesmo tipo nunca sai tres dias seguidos. O bastidor aparece no maximo uma vez a cada quatro posts, o carrossel nao se repete nos dois posts seguintes e a enquete aparece no maximo uma vez a cada sete posts.

📐 Arquitetura da Solucao

[ Schedule Trigger ] ──► (Dispara as 08h / Cron 24/7 na VPS)
         │
         ▼
[ Wait ] ──► (Espera sorteada de 10 a 160 minutos: o horario muda a cada dia)
         │
         ▼
[ GitHub REST API ] ──► (Lista os repositorios publicos; projeto novo entra sozinho na fila)
         │
         ▼
[ Notion Database ] ──► (Le os temas ja publicados para evitar repeticao)
         │
         ▼
[ Code: tema do dia ] ──► (Sorteia tipo, area, formato, tom, gancho e tamanho)
         │
         ▼
[ HTTP: README do projeto ] ──► (Material do post de bastidor)
         │
         ▼
[ Google Gemini Flash ] ──► (Saida estruturada TITULO / BUSCA / POST, mais SLIDES ou ENQUETE)
         │        └─ se falhar ──► [ Gemini Flash Lite ] (modelo de reserva)
         ▼
[ Code: validacao ] ──► (Remove markdown e caracteres especiais, valida tamanho, slides e enquete)
         │
         ├─ foto ──────► [ Pixabay ] ► [ Escolha da foto ] ► [ Download binario ] ► [ LinkedIn: texto + imagem ]
         ├─ carrossel ─► [ Documents API: iniciar envio ] ► [ Code: PDF ] ► [ Envio do PDF ] ► [ Posts API: documento ]
         ├─ enquete ───► [ Posts API: enquete ]
         └─ texto e bastidor ► [ LinkedIn: so texto ]  ◄── reserva se carrossel ou enquete falharem
         │
         ▼
[ Historico + Notion ] ──► (Grava fotos, temas e tipos usados e registra o post como "Concluido")

🛠️ Desafios Tecnicos & Solucoes de Engenharia

• Processamento de Midia Binaria (Buffer Stream): A API do LinkedIn rejeita URLs externas diretas de imagem. A solucao utiliza uma camada HTTP intermediaria que faz o download dos bytes reais do CDN do Pixabay para a memoria temporaria do n8n, transmitindo os dados via multipart/form-data.

• Carrossel em PDF sem navegador: O n8n roda em container sem Chromium e sem bibliotecas externas no no de codigo. O PDF e montado byte a byte em JavaScript puro: paginas de 1080x1350, fontes padrao do PDF com codificacao WinAnsi para manter os acentos, e uma tabela de larguras de letra para quebrar as linhas e reduzir o tamanho da fonte ate o texto caber. O arquivo vai para a Documents API do LinkedIn e o post e criado pela Posts API.

• Saida Estruturada e Validacao: O LLM responde em um formato fixo (TITULO / BUSCA / POST, com um bloco SLIDES ou ENQUETE quando o tipo pede). Um no de codigo separa os campos, remove markdown e caracteres que quebram a publicacao, corta textos longos e interrompe a execucao se o post vier vazio ou sem titulo.

• Rotas de Reserva: Se o modelo principal do Gemini falhar por cota ou alta demanda, um segundo modelo escreve o post. Se os slides ou a enquete vierem fora do formato, ou se o LinkedIn recusar o carrossel ou a enquete, o post do dia sai so com o texto, sem as frases que mencionam slides ou votacao.

• Conteudo Autoalimentado: A lista de projetos vem da API do GitHub a cada execucao, filtrando forks, repositorios arquivados e projetos sem atividade no ultimo ano. O post de bastidor usa somente fatos presentes no README do repositorio sorteado.

• Algoritmo Anti-Duplicacao: Os IDs das ultimas 300 fotos publicadas ficam no static data do workflow e sao descartados na escolha; os temas ja publicados (Notion + memoria interna) vao no prompt, e as ultimas areas, formatos, aberturas, tipos e projetos nao sao sorteados de novo. O historico so e gravado em execucoes agendadas, nao em testes manuais.

⚙️ Configuracao

1. Importe o arquivo `PostEngineAI.json` no n8n.
2. Crie as credenciais e selecione cada uma nos nos correspondentes:
   • Gemini: tipo Header Auth, Name `x-goog-api-key`, Value = chave do Google AI Studio.
   • Pixabay: tipo Query Auth, Name `key`, Value = chave da API do Pixabay.
   • Notion: integracao com acesso a base de posts.
   • LinkedIn: OAuth2 com "Organization Support" e "Legacy" desligados; o app precisa dos produtos "Share on LinkedIn" e "Sign In with LinkedIn using OpenID Connect". A mesma credencial e usada nos nos HTTP do carrossel e da enquete.
3. Ajuste a base do Notion, a pessoa do LinkedIn (nos nos de publicacao e nos nos HTTP, no formato `urn:li:person:ID`), o usuario do GitHub (no "Buscar projetos no GitHub" e constante `DONO` no no "Montar tema do dia") e o fuso horario da instancia (`GENERIC_TIMEZONE`).
4. Ative o workflow.

Nenhuma chave fica no JSON: o arquivo guarda apenas o nome e o ID das credenciais.

---

## 🇺🇸 ENGLISH

An end-to-end automation architecture built with n8n for autonomous creation, validation, media generation, and daily publishing of content to LinkedIn, integrating Artificial Intelligence (Google Gemini), automated image retrieval (Pixabay API), GitHub project discovery, PDF carousel generation, and history tracking via Notion.

📌 Project Overview
This project solves the challenge of maintaining consistent publication activity on LinkedIn without requiring daily manual input, and without every post coming out of the same mold.

The workflow fires every day at 8 AM and waits for a random interval, so the post goes out at a different time each day, between 8:10 and 10:40 AM. It then reads the author's public GitHub repositories and the topics already published in Notion, and draws the post type, area, format, tone and text length. Gemini writes the content in a structured format, a code node validates and cleans the response, and the post follows the route of the drawn type. Finally, the workflow stores its internal history and logs the post in Notion.

🎲 Post Types

| Type | Draw weight | What gets published |
|---|---|---|
| Photo | 35% | Text with a Pixabay photo that has not been used before, with photographer credit |
| Text only | 22% | Text without an image, usually shorter |
| Behind the scenes | 18% | Text about one of the author's projects, written from the repository README, with a link to the code |
| Carousel | 17% | Short caption plus a 5 to 7 slide PDF generated inside n8n itself |
| Poll | 8% | Short text plus a poll with 3 or 4 options, open for 7 days |

The same type never comes out three days in a row. Behind-the-scenes posts appear at most once every four posts, a carousel is not repeated within the next two posts, and polls appear at most once every seven posts.

📐 Solution Architecture

[ Schedule Trigger ] ──► (Fires at 8 AM / 24/7 Cron on VPS)
         │
         ▼
[ Wait ] ──► (Random wait of 10 to 160 minutes: the time changes every day)
         │
         ▼
[ GitHub REST API ] ──► (Lists public repositories; a new project joins the queue on its own)
         │
         ▼
[ Notion Database ] ──► (Reads published topics to avoid repetition)
         │
         ▼
[ Code: daily topic ] ──► (Draws type, area, format, tone, hook and length)
         │
         ▼
[ HTTP: project README ] ──► (Source material for behind-the-scenes posts)
         │
         ▼
[ Google Gemini Flash ] ──► (Structured TITULO / BUSCA / POST output, plus SLIDES or ENQUETE)
         │        └─ on failure ──► [ Gemini Flash Lite ] (fallback model)
         ▼
[ Code: validation ] ──► (Strips markdown and special characters, validates length, slides and poll)
         │
         ├─ photo ─────► [ Pixabay ] ► [ Photo selection ] ► [ Binary download ] ► [ LinkedIn: text + image ]
         ├─ carousel ──► [ Documents API: init upload ] ► [ Code: PDF ] ► [ PDF upload ] ► [ Posts API: document ]
         ├─ poll ──────► [ Posts API: poll ]
         └─ text and behind the scenes ► [ LinkedIn: text only ]  ◄── fallback if carousel or poll fail
         │
         ▼
[ History + Notion ] ──► (Stores used photos, topics and types and logs the post as "Done")

🛠️ Technical Challenges & Engineering Solutions

• Binary Payload Processing (Buffer Stream): The LinkedIn API rejects direct external image URLs. The solution implements an intermediate HTTP download layer that streams raw bytes from Pixabay's CDN to n8n's temporary memory, transmitting data via multipart/form-data.

• PDF Carousel Without a Browser: n8n runs in a container with no Chromium and no external libraries in the code node. The PDF is assembled byte by byte in plain JavaScript: 1080x1350 pages, standard PDF fonts with WinAnsi encoding to keep accented characters, and a glyph width table to wrap lines and shrink the font size until the text fits. The file is sent to the LinkedIn Documents API and the post is created through the Posts API.

• Structured Output and Validation: The LLM answers in a fixed format (TITULO / BUSCA / POST, with a SLIDES or ENQUETE block when the type requires it). A code node splits the fields, strips markdown and characters that break publishing, trims long texts and stops the execution if the post comes back empty or without a title.

• Fallback Routes: If the main Gemini model fails due to quota or high demand, a second model writes the post. If the slides or the poll come back malformed, or if LinkedIn rejects the carousel or the poll, the day's post goes out as text only, without the sentences that mention slides or voting.

• Self-Feeding Content: The project list comes from the GitHub API on every run, filtering out forks, archived repositories and projects with no activity in the last year. Behind-the-scenes posts use only facts found in the README of the drawn repository.

• Anti-Duplication Algorithm: The IDs of the last 300 published photos are kept in the workflow static data and discarded during selection; previously published topics (Notion + internal memory) are sent in the prompt, and the latest areas, formats, openings, types and projects are not drawn again. History is only stored on scheduled executions, not on manual tests.

⚙️ Setup

1. Import `PostEngineAI.json` into n8n.
2. Create the credentials and select each one in its nodes:
   • Gemini: Header Auth type, Name `x-goog-api-key`, Value = Google AI Studio key.
   • Pixabay: Query Auth type, Name `key`, Value = Pixabay API key.
   • Notion: integration with access to the posts database.
   • LinkedIn: OAuth2 with "Organization Support" and "Legacy" turned off; the app needs the "Share on LinkedIn" and "Sign In with LinkedIn using OpenID Connect" products. The same credential is used by the carousel and poll HTTP nodes.
3. Adjust the Notion database, the LinkedIn person (in the publishing nodes and in the HTTP nodes, as `urn:li:person:ID`), the GitHub user (in "Buscar projetos no GitHub" and the `DONO` constant in the "Montar tema do dia" node) and the instance timezone (`GENERIC_TIMEZONE`).
4. Activate the workflow.

No keys are stored in the JSON: the file only holds credential names and IDs.

---

## 🧰 Tech Stack

• Orchestrator / Orquestrador: n8n Self-Hosted (Hostinger Linux VPS)
• AI Engine / Motor de IA: Google Gemini API (Gemini 3.5 Flash, com Flash Lite de reserva / with Flash Lite as fallback)
• Media Provider / Midias: Pixabay REST API + PDF gerado em codigo / code-generated PDF
• Project Source / Fonte de projetos: GitHub REST API
• CMS & Database: Notion API
• Target Platform / Destino: LinkedIn API (Posts API + Documents API)

---

## 👨‍💻 Autor / Author

Leonardo Magalhaes
Desenvolvedor & Entusiasta em Automacao de Processos | Process Automation & Software Engineering
GitHub: [https://github.com/leomagalhaesti](https://github.com/leomagalhaesti)
