# Auditoria dos materiais publicados — Físico-Química II 2026.2

**Disciplina:** Físico-Química II · 2026.2 · Prof. Dr. Rodrigo José de Oliveira (UEPB/CCT/Química · PCM Lab)
**Escopo auditado:** todo o código publicado em `https://pcmlab-uepb.github.io/fq2-materiais/`
(repositório local `~/fq2-site`: hub + 6 aulas × `index.html` / `apostila*.html` / `quiz.html` / `slides*.pdf`, `assets/site.css`, `robots.txt`).
**Commit auditado:** `23df45c` (antes) → **`dbea80d` (consertos publicados)**.
**Data:** 18/10/2026.

## Método (o que é medida e o que é leitura de código)

| verificação | como foi feita |
|---|---|
| links internos | extração de **todos** os `href`/`src`/`url()` dos 24 HTML + 1 CSS; resolução de caminho e de âncora; conferência de existência do arquivo e do `id`/`name` de destino |
| links publicados | `curl` HTTP(S) nas 36 URLs do site + nas 6 videoaulas + nos 6 Google Forms |
| HTML | parser próprio (balanço de tags), busca de `id` duplicado e de `img` sem `alt` |
| JS | `node --check` em **todo** `<script>` inline, inclusive os embutidos em base64 (`srcdoc`) das apostilas |
| layout do menu (**B**) | **medida em navegador real**: Chromium 1243 headless via Playwright, 5 viewports (1920×1080, 1366×768, 1024×768, 768×1024, 390×844) — `getBoundingClientRect`, `getComputedStyle` e `elementFromPoint`; mais teste de clique real em **todos** os 75 links de menu |
| produção | as mesmas medidas refeitas **na URL no ar**, depois do push |

> **Limite declarado:** não houve inspeção visual (o agente não vê imagem). Toda conclusão de layout vem de **geometria medida** (retângulos/estilos computados) e de `elementFromPoint`, não de "olhar a tela". As capturas ficaram em disco para conferência humana (ver §E).

---

## 0. Resultado

- **Defeito 1 (links internos)** — **causa-raiz encontrada e consertada.** Não existe nenhum `href` quebrado no repositório (0 de 166 referências internas). O que "não abria" eram **os 21 links do menu lateral da apostila da aula 1 quando outra aba (Quiz/Simulação) estava ativa**: o alvo fica dentro de um `.view` com `display:none`, o clique não rola nada.
- **Defeito 2 (barra de menu translúcida sobre os slides)** — **causa-raiz encontrada e consertada.** A regra CSS `nav{...}` (barra lateral de 300 px, `height:100vh`, `position:sticky`, fundo `rgba(10,18,15,.55)`) alcançava também o `<nav class="abas">` do painel de interativos, criando **uma coluna translúcida de 300 × 100vh fixada sobre o conteúdo**; e a seção de interativos estava **fora do `<main>`**, virando uma 3ª coluna flex que **espremia os slides para 200–519 px**. Medido antes/depois em 5 viewports.
- **Mais grave que os dois** (achado da varredura C): em 6 das 11 apostilas, **43 páginas do deck não têm entrada própria no menu** (aula 4: 5 entradas para 14 páginas; aula 2: 13 para 27). Não é link quebrado — é índice incompleto herdado do gerador (uma entrada por *título distinto*, com rótulo em faixa "4–5"). **Não consertado** (mudaria o material gerado; ver §A.5/D).
- **Publicado:** commit **`dbea80d`** (11 arquivos, +51 −42 linhas), Pages `built`, e os dois consertos **revalidados na URL no ar**.
- **Nada foi alterado em texto, fórmula ou número.**

---

## A) Links internos

### A.1 Inventário

| item | valor |
|---|---|
| arquivos varridos | 25 (24 HTML + `assets/site.css`) |
| referências extraídas | 327 |
| — internas (arquivo/âncora) | 166 |
| — externas (YouTube) | 6 |
| — ignoradas (`data:` de figura embutida) | 155 |
| **internas quebradas (arquivo ausente)** | **0** |
| **âncoras quebradas (`#id` sem destino)** | **0** |
| HTML órfão (não alcançável a partir do hub) | 0 |
| `id` duplicado | 0 |
| `<img>` sem `alt` | 0 (todas as 155 figuras têm `alt`) |

Todas as âncoras dos menus (`#p1…#p27`, `#c0…#c19`) existem no próprio arquivo. `assets/site.css` e `../` (o "voltar") resolvem.

### A.2 O defeito 1 — os links do menu morriam fora da aba "Aula" (aula 1)

A apostila da aula 1 tem três abas: **Aula / Quiz interativo / Simulação**. Os 21 itens do menu lateral apontam para `#c0…#c19`, que ficam **dentro de `<div class="view" id="v-aula">`**. Ao ativar outra aba, o CSS `.view{display:none}` esconde essa div — e o clique no menu passa a não abrir nada (o navegador não rola até um elemento oculto). *Só a aula 1 tem esse mecanismo de abas; as apostilas 2–6 têm os interativos embutidos numa seção própria.*

**Prova — clique real em Chromium, 1366×768:**

| clique | ANTES (`23df45c`) | DEPOIS (`dbea80d`) |
|---|---|---|
| `#c3` com aba Quiz ativa | `scrollY=0`, alvo invisível, `elementFromPoint` = `nav` (**link morto**) | `scrollY=4290`, `alvoTop=0`, visível, frente = `h2` (ok) |
| `#c10` idem | `scrollY=0`, morto | `scrollY=13320`, visível (ok) |
| `#c19` idem | `scrollY=0`, morto | `scrollY=23094`, visível (ok) |

**Conserto** (`aula1-fundamentos/apostila.html`, +14 linhas de JS): ao clicar em qualquer link do menu, se o alvo estiver dentro de um `.view` inativo, a aba desse `.view` é **reativada antes** de o navegador rolar até a seção (e o estado visual dos botões de aba é atualizado junto). Nada mais mudou.

### A.3 Verificação do estado normal (nenhuma regressão)

Clique real em **todos** os links de menu das 6 apostilas principais, 1366×768, **depois** do conserto:

| apostila | links de menu testados | não alcançam o alvo |
|---|---|---|
| aula1-fundamentos | 21 | 0 |
| aula2-gibbs-helmholtz | 13 | 0 |
| aula3-equilibrio-fases | 6 | 0 |
| aula4-transicoes-fase | 5 | 0 |
| aula5-misturas | 10 | 0 |
| aula6-equilibrio-quimico | 20 | 0 |
| **total** | **75** | **0** |

### A.4 Links que os alunos recebem — verificação HTTP (site no ar)

Todas as 36 URLs publicadas respondem **200** com `content-type` coerente — **antes e depois** (nada mudou de caminho):

| URL (relativa ao site) | HTTP | content-type |
|---|---|---|
| `aula{1,2,3,4,5,6}/index.html` | 200 | text/html |
| `aula{1,2,3,4,5,6}/slides.pdf` | 200 | application/pdf |
| `aula3,4/slides-pratique.pdf`, `aula5/slides-51.pdf`, `aula5/slides-sistemas-binarios.pdf` | 200 | application/pdf |
| `aula{1..6}/apostila.html` | 200 | text/html (2,3 a 9,4 MB) |
| `aula3,4/apostila-pratique.html`, `aula5/apostila-51.html`, `aula5/apostila-sistemas-binarios.html` | 200 | text/html |
| `aula{1..6}/quiz.html` | 200 | text/html |
| `assets/site.css` | 200 | text/css |
| `robots.txt` | 200 | text/plain |

As 6 postagens do Classroom apontam para `.../<aula>/slides.pdf` e `.../<aula>/apostila.html` (+ vídeo) — **todos 200**. Nenhum 404 no material publicado.

### A.5 O que existe de verdade atrás de "não abre" (2º achado, **não consertado**)

Em 6 apostilas o menu é **deduplicado por título**: páginas consecutivas com o mesmo `h2` viram uma única entrada com rótulo em faixa (ex.: `#p5 <em>5–14</em>`). Efeito: há páginas do deck **sem entrada própria no menu** (alcançáveis só rolando). Em uma delas o menu salta de 4 para 6.

| apostila | entradas no menu | páginas do deck | páginas sem entrada |
|---|---|---|---|
| aula4-transicoes-fase | 5 | 14 | **9** (p6…p14) |
| aula2-gibbs-helmholtz | 13 | 27 | **14** (p4…p13, p17, p19, p24, p26) |
| aula3-equilibrio-fases | 6 | 13 | **7** (p3, p4, p5, p7, p10, p11, p12) |
| aula6-equilibrio-quimico | 20 | 26 | **6** (p3, p10, p12, p13, p18, p19) |
| aula5-misturas/apostila-51.html | 9 | 15 | **6** (p4, p9, p11, p13, p14, p15) |
| aula5-misturas/apostila-sistemas-binarios.html | 11 | 12 | **1** (p5 — o menu vai de 4 para 6) |
| aula1, aula5/apostila, 3 `apostila-pratique` (aulas 3,4,5) | — | — | 0 (índices completos) |

Isso **não** é link quebrado (o alvo de cada entrada existe), mas é a explicação mais provável de parte da percepção "alguns links não abrem" — nota-se principalmente na aula 4, onde o menu mostra 5 itens para 14 páginas. **Não consertei** porque as entradas são geradas por script a partir dos títulos do material (mexer nisso é regerar conteúdo, não corrigir código). Ver §D.

### A.6 Órfãos / duplicatas (a pergunta do enunciado)

**Nenhum arquivo é órfão.** Os arquivos "suspeitos" estão todos linkados nas páginas de aula:

| arquivo | quem aponta para ele | veredito |
|---|---|---|
| `aula3-equilibrio-fases/apostila-pratique.html` | `aula3-equilibrio-fases/index.html` | **não é sobra** |
| `aula4-transicoes-fase/apostila-pratique.html` | `aula4-transicoes-fase/index.html` | **não é sobra** |
| `aula5-misturas/apostila-pratique.html` | `aula5-misturas/index.html` | **não é sobra** |
| `aula5-misturas/apostila-51.html` | `aula5-misturas/index.html` | **não é sobra** |
| `aula5-misturas/apostila-sistemas-binarios.html` | `aula5-misturas/index.html` | **não é sobra** |
| `robots.txt` | nenhum link (proposital) | infraestrutura, esperado |

Observação: esses 5 arquivos extras são alcançáveis **apenas pelo índice da própria aula** (o hub aponta só para `apostila.html`) — 2 cliques a partir do hub. Se a intenção era que o aluno visse o deck "Pratique"/"5.1"/"sistemas binários" direto do hub, falta o cartão; se a intenção é só dentro da aula, está correto.

---

## B) A barra de menu translúcida sobre os slides

### B.1 Causa-raiz (dois mecanismos independentes, os dois confirmados por medida)

1. **Vazamento do seletor `nav`.** O CSS de cada apostila define a barra lateral com o seletor de elemento
   `nav{width:300px;flex:0 0 300px;background:rgba(10,18,15,.55);position:sticky;top:0;height:100vh;overflow:auto}`.
   Nas apostilas 2–6 existe **um segundo `<nav>`** — o `<nav class="abas">` do painel "Interativos desta aula" (botões *Simulação / Quiz*).
   Como `nav` é seletor de elemento, ele também pegou esse `<nav>`: a barra de abas virou **300 px de largura × 100vh de altura, `position:sticky`, com fundo translúcido** — exatamente a "barra de menu transparente" que fica **presa sobre o conteúdo** enquanto se rola a página (o `height:100vh` + `sticky` mantém a barra colada no topo durante todo o scroll).
   Confirmado no navegador: `nav.abas` computava `300×1080`, `position:sticky`, `background:rgba(10,18,15,0.55)`.
2. **A seção de interativos ficou fora do `<main>`.** Nas 5 apostilas com painel embutido, `</main>` fecha **antes** de `<section id="interativos-embutidos">`. Com `body{display:flex}`, essa seção virou um **terceiro item flex**: como os itens flex esticam (`align-items:stretch`), ela ficou com a mesma altura da página inteira (ex.: 10102 px) e o `<nav class="abas">` sticky pôde ficar pinado **do começo ao fim do documento**; e como o `main` compartilhava a linha com ela, **os slides foram espremidos** — medido: **288 px** de largura útil a 1366 px e **238–519 px** conforme a aula (aula 3 a 1366 px ficou com **200 px**).

### B.2 Medidas — antes × depois (Chromium headless, medida real de layout)

Apostila da **aula 2** (`aula2-gibbs-helmholtz/apostila.html`):

| viewport | largura do `main` (os slides) | `<nav class="abas">` (x, largura, altura, posição) | fundo do `nav.abas` | barra lateral | overflow horizontal |
|---|---|---|---|---|---|
| 1920×1080 | **519 → 1150** | (819, 300, 1080, `sticky`) → (330, 1090, **40**, `static`) | `rgba(10,18,15,.55)` → **transparente** | 300×1080 `sticky` (inalterada) | não → não |
| 1366×768 | **288 → 1066** | (588, 300, 768, `sticky`) → (330, 1006, **40**, `static`) | idem | 300×768 `sticky` | não → não |
| 1024×768 | **288 → 724** | (588, 300, 768, `sticky`) → (330, 664, **40**, `static`) | idem | 300×768 `sticky` | não → não |
| 768×1024 | 768 → 768 | (0, 768, 80, `static`) → (30, 708, 40, `static`) | idem | 768×615 `static` | não → não |
| 390×844 | 390 → 390 | (0, 390, 80, `static`) → (30, 330, 40, `static`) | idem | 390×674 `static` | não → não |

Apostila da **aula 6** — mesmo padrão: `main` **489 → 1150** (1920), **238 → 1066** (1366), **238 → 724** (1024).

Todas as 5 apostilas com painel embutido:

| arquivo | 1920×1080 | 1366×768 | 1024×768 |
|---|---|---|---|
| aula2-gibbs-helmholtz/apostila.html | 519 → 1150 | 288 → 1066 | 288 → 724 |
| aula3-equilibrio-fases/apostila.html | 475 → 1150 | **200 → 1066** | **200 → 724** |
| aula4-transicoes-fase/apostila.html | 476 → 1150 | 265 → 1066 | 265 → 724 |
| aula5-misturas/apostila.html | 641 → 1150 | 248 → 1066 | 248 → 724 |
| aula6-equilibrio-quimico/apostila.html | 489 → 1150 | 238 → 1066 | 238 → 724 |

**Controle (prova de que o conserto é cirúrgico):** nos arquivos que **não** têm o painel de interativos (`aula1/apostila.html`, `aula3/apostila-pratique.html`, `aula5/apostila-51.html`), a geometria é **byte a byte a mesma** antes e depois (`main` 1066/724/390, barra lateral (300,768,`sticky`) — `iguais=True`).

### B.3 Conserto aplicado (identidade visual preservada)

1. `nav{…}` → **`body>nav{…}`** (e igual no `@media`) nas **11 apostilas** — a barra lateral continua exatamente igual (mesma largura, cor, `sticky`); o que deixa de acontecer é a regra contaminar outro `<nav>`.
2. `<section id="interativos-embutidos">` passa a ficar **dentro do `<main>`** (o `</main>` foi movido para o fim do corpo, nas 5 apostilas). O painel de interativos volta a ter a largura do conteúdo e **para de esticar por 100vh**.
3. `:root` das 5 apostilas ganhou os tokens que o painel usava mas **não existiam** (`--line`, `--card`, `--accent` — sem eles os botões/iframes do painel ficavam sem borda e sem fundo) e a aba ativa passou a usar `--ink` no lugar de `#fff` (contraste **1,68:1 → 10,57:1**).

**Efeito medido depois do conserto (produção):** `nav.abas` = **40 px de altura, estático, fundo transparente** em todos os viewports; `main` com a largura plena (1150/1066/724/768/390); `#interativos-embutidos` **dentro do `main`**; sem rolagem horizontal (`scrollWidth == viewport`) em nenhum viewport.

### B.4 Conteúdo inalcançável? E o rodapé?

- **Título / primeiro slide:** no desktop nunca ficaram escondidos (a barra lateral vive em coluna própria, não sobre o conteúdo — confirmado por `elementFromPoint` na geometria). O que a barra translúcida cobria era **o painel de interativos** (e a faixa onde os slides "deveriam" estar, já que `main` estava espremido): com a página rolada até o fim, os pontos medidos dentro do retângulo do `nav.abas` batiam em `nav.abas`/`button.aba` — a barra estava **na frente** do conteúdo ali. Depois do conserto, os mesmos pontos batem no próprio conteúdo (`p.tag`, `section…`).
- **Rodapé:** nenhum menu cobre o rodapé em nenhum viewport (a barra lateral e a de abas não se sobrepõem ao `footer.site`; nas apostilas não há `footer`).
- **Celular — pendente declarado:** a `@media(max-width:900px)` deixa a barra lateral como **bloco estático no topo**: a 390 px ela mede **1401 px de altura na aula 1** (829 px na aula 6). Ou seja, no celular o aluno precisa rolar ~2–3 telas de menu antes do título/primeiro slide. Não estava escondido e o professor não relatou isso, mas é um ponto real de usabilidade — **não mexi** (a correção seria menu colapsável, mudança de design; ver §D).

---

## C) Varredura geral do código

### C.1 HTML

| checagem | resultado |
|---|---|
| balanço de tags (parser) nos 24 HTML, **depois** do conserto | **todos OK** (nenhuma tag aberta ou fechada a mais) |
| `id` duplicado | 0 |
| `<img>` sem `alt` / com `alt` vazio | 0 / 0 (155 figuras) |
| `<nav class="abas">` com `data-alvo` apontando para iframe inexistente | 0 |
| páginas de quiz: `main#main`, `aside`/`header` fechados | OK |

### C.2 JavaScript

| checagem | resultado |
|---|---|
| `node --check` em **todo** `<script>` inline dos 24 HTML **e** nos 12 payloads base64 (`srcdoc` das apostilas 1–6) | **0 erro de sintaxe** |
| `console.error` / `console.warn` / `debugger` | 0 |
| `getElementById`/`querySelector` de id inexistente | 0 (os únicos achados na varredura eram `getElementById('int-' + k)` e classes aplicadas em tempo de execução — falso positivo, conferido no código) |
| `fetch()` relativo (quebraria em `file://`/base) | 0 |
| scripts externos (`<script src>`) | 0 (tudo inline; as apostilas são autossuficientes) |
| listener amarrado a elemento inexistente | 0 |

### C.3 CSS

| item | resultado |
|---|---|
| arquivo de CSS externo | 1 (`assets/site.css`, 31 regras) |
| fonte/caminho de asset inexistente | 0 (tipografia é 100% system-stack: `DejaVu Sans`, `Verdana`, `system-ui` — nada a carregar) |
| `z-index` absurdo | 0 (não há `z-index` em nenhum arquivo) |
| **tokens CSS usados e não definidos** | `--line`, `--card`, `--accent` no painel de interativos das apostilas 2–6 → **consertado** (§B.3) |
| regras CSS órfãs | só aparentes: `.fb.good`, `.mod.on`, `.rev .tag.ok` etc. são classes **criadas em runtime pelo JS** — não são órfãs |
| contraste (WCAG, mínimo 4,5:1) | |

Contraste medido (razão calculada sobre as cores reais do arquivo):

| texto | antes | depois | veredito |
|---|---|---|---|
| aba ativa do painel (era `#fff` sobre o accent) | **1,68:1** | **10,57:1** (`--ink` sobre `#7fd4e8`) | corrigido |
| aba inativa (`--cyan` sobre fundo `#16241f`) | 9,56:1 | 9,56:1 | ok |
| texto `--chalk` sobre fundo da apostila | 14,07:1 | 14,07:1 | ok |
| `.meta` (opacidade .7) sobre fundo | 7,70:1 | 7,70:1 | ok |
| hub: link accent sobre cartão branco | 6,40:1 | 6,40:1 | ok |
| hub: `.tag`/rodapé (muted sobre claro) | 5,47:1 / 5,14:1 | idem | ok |
| quiz: chalk e cyan sobre o gradiente verde | 6,96:1 / 4,63:1 | idem | ok |

### C.4 Órfãos e duplicatas

- HTML: **0 órfãos** (ver §A.6).
- CSS/JS/asset: 0 arquivos extras; `robots.txt` é o único não referenciado por link (proposital).
- **Duplicatas**: as 5 apostilas "extra" (Pratique/5.1/sistemas binários) **não** são cópias duplicadas umas das outras — os decks e os `slides*.pdf` correspondentes são distintos por conteúdo; nenhuma duplicata byte a byte foi encontrada no repositório.
- Observação (não é defeito): peso dos arquivos — a apostila da aula 1 tem **9,4 MB** (figuras em base64 embutidas). Abre offline (é o objetivo), mas é pesada em rede móvel. Cabe no Pages.

### C.5 Links externos

| destino | resposta |
|---|---|
| `https://youtu.be/S_HuP3SU4oU`, `3Lu2YnmUb5Q`, `Rv1y02xJD54`, `8K5DyAdvjjo`, `HYLnLgFkZf4`, `g_JhgoxlT_U` (os 6 links de videoaula das páginas de aula) | **200** (os 6) |
| 6 Google Forms (quizzes da turma) via `curl` | **401** — é o *portão de login* do Google para acesso anônimo, **não** link quebrado: no navegador a página abre (status 200) mostrando "Faça login / Continuar para o Google Formulários". O formulário é restrito à turma, como deve ser. |

### C.6 O que NÃO deu para verificar (declarado, não inventado)

1. **Aparência visual** (pixel a pixel): o agente não vê imagem; as conclusões de layout são geométricas. Capturas estão em disco (§E) para conferência humana.
2. **Comportamento dentro do Google Classroom / Forms autenticados**: sem sessão da turma aqui; verifiquei o *estado registrado* da turma (`~/fq2-resgate/cache/classroom_DEPOIS_aviso.json` + relatórios do resgate) e que cada URL ali referida responde 200 — não re-consultei a API do Classroom.
3. **Interativos dentro dos `srcdoc`** (quiz/simulação embutidos): auditei sintaxe e o HTML deles (decodificados dos base64); não cliquei em cada botão de cada um dos 12 payloads.
4. **O que o professor viu exatamente**: os dois defeitos relatados foram reproduzidos por mecanismo; a hipótese de 404 está **descartada** (nada 404 no ar).

---

## D) Pendências (com o motivo)

| # | pendência | gravidade | por que ficou |
|---|---|---|---|
| D1 | **43 páginas de deck sem entrada própria no menu** (aula 2: 14; aula 4: 9; aula 3: 7; aula 6: 6; aula5/51: 6; aula5/sistemas-binários: 1) | ruim (usabilidade) | o menu é **gerado** com uma entrada por título distinto (entradas em faixa, ex. "5–14"); corrigir = regerar o índice do material, não corrigir código. Recomendação: regerar com **uma entrada por página**. |
| D2 | **Menu lateral no celular é um bloco de 829–1401 px** antes do título | ruim (usabilidade) | exigiria menu colapsável (mudança de design). O professor não relatou; resolvi não mexer no que funciona. |
| D3 | **Descrição das 6 videoaulas no YouTube não contém nenhum link** do site (verificado pela API: nenhuma ocorrência de `http` ou `pcmlab` nas 6 descrições; só o aviso beta + um parágrafo de apresentação) | ruim (alcance do material) | o enunciado supunha que a descrição continha os links — **não contém**. Alterar descrição pública de vídeo no YouTube é decisão editorial do professor (e a ferramenta disponível só *acrescenta* texto no início), então não mexi. Se quiser, é 1 comando por vídeo (`tools/youtube_video.py editar-descricao`). |
| D4 | Os 5 decks extras (Pratique/5.1/sistemas binários) só são alcançáveis pelo índice da aula, não do hub | cosmético | pode ser intencional; depende do que o professor quer expor no hub. |
| D5 | Apostila da aula 1 com 9,4 MB | cosmético | inerente às figuras em base64 (funciona offline). Reduzir exigiria recomprimir as figuras = mexer no material. |

---

## E) Evidências e como reproduzir

**Conserto publicado:** commit `dbea80d` (`23df45c..dbea80d main -> main`); Pages `builds/latest` = `built` para esse commit; `ls-remote` confere `dbea80d…` como `refs/heads/main`.
**Sandbox:** etapa `20261018-140642-d554dfb44a23` (journal em `~/.local/share/opencode/sandbox/journal.jsonl`), 11 arquivos, +51 −42; rollback byte a byte disponível em `~/.local/share/opencode/sandbox/rollback/20261018-140642-d554dfb44a23`.

**Prova em produção** (medida na URL no ar, depois do push): `aula2…/apostila.html` e `aula6…/apostila.html` em 5 viewports → `main` 1150/1066/724/768/390, `nav.abas` 40 px estático transparente, `#interativos-embutidos` dentro do `main`, `scrollWidth == viewport`; `aula1…/apostila.html` com a aba Quiz ativa → clique no menu rola para `scrollY` 4290 / 13320 / 23094 com o alvo visível e na frente (`section.cena`).

**Dados brutos e scripts** (fora do repositório, em `~/.hermes/profiles/pesquisador-academico-em-quimica/cache/scratch/`):

| arquivo | conteúdo |
|---|---|
| `audit_links.py` → `audit/links.json` | todas as 327 referências, resolução, quebradas, órfãos |
| `audit_jscss.py` → `audit/jscss.json`, `audit/jscss.txt` | `node --check`, JS/CSS, payloads base64 |
| `render_one.py` + `audit/geom_antes.jsonl` / `audit/geom.jsonl` | geometria por página × viewport antes/depois |
| `anchor_test.py` → `audit/anchors_depois.jsonl` (75 links) | clique real em todos os links de menu |
| `verify_a.py` → `audit/verify_a_antes.json` / `_depois.json` | o defeito 1 antes/depois |
| `probes.py` → `audit/probes_antes.json` / `probes_depois.json` | rede de colunas, `nav.abas`, cobertura no fim da página |
| `verify_producao.py` → `audit/producao.json` | as mesmas medidas na URL no ar |
| `audit/tabela_viewports.txt` | tabelas de viewport em texto |
| `audit/shots/*.png` | 110 capturas (após o conserto) para conferência humana |
| `audit/http_live.json` | HTTP de todas as URLs publicadas |

---

## F) Pendências D1–D3 fechadas — o que foi feito, a prova e o commit (18/10/2026)

**Commit do site:** **`02e451d`** (`dbea80d..02e451d main -> main`), 11 arquivos, **+732 −17**.
**Sandbox:** etapa `20261018-142827-75b77238b759` (site, 28 operações) e `20261018-142957-b6ed3cb4fa2b` (ferramenta do YouTube, 7 operações); rollback em `~/.local/share/opencode/sandbox/rollback/<etapa>/`.
**Método:** tudo por **comportamento medido em Chromium real** (Playwright, `TMPDIR=/tmp`), não por inspeção — clique real em item de menu, `getBoundingClientRect`, `getComputedStyle`, `elementFromPoint`, `document.activeElement` e `Tab`/`Enter`/`Escape` de verdade. **O agente não vê imagem**: não houve inspeção visual, e isso segue valendo.

### F.1 D1 — 43 páginas de deck sem entrada própria no menu → índice 1:1

**Antes:** o menu era deduplicado por título, com rótulo em faixa (`<em>5–14</em>`); 43 páginas do deck não tinham entrada.
**Depois:** o índice foi **regerado com uma entrada por página** (`<a href="#pN"><span>h2 da página</span><em>N</em></a>`), na ordem canônica. **Nenhum título foi inventado**: o rótulo é o próprio `<h2>` que a página já mostra, e o `<em>` é o número que a página já exibe (`página N`).

| apostila | entradas antes | entradas depois | páginas de deck | páginas sem entrada |
|---|---|---|---|---|
| aula2-gibbs-helmholtz | 13 | **27** | 27 | 14 → **0** |
| aula3-equilibrio-fases | 6 | **13** | 13 | 7 → **0** |
| aula4-transicoes-fase | 5 | **14** | 14 | 9 → **0** |
| aula5-misturas/apostila-51 | 9 | **15** | 15 | 6 → **0** |
| aula5-misturas/apostila-sistemas-binarios | 11 | **12** | 12 | 1 → **0** |
| aula6-equilibrio-quimico | 20 | **26** | 26 | 6 → **0** |
| **total** | **64** | **107** | 107 | **43 → 0** |

As outras 5 apostilas já tinham índice 1:1 e **não foram tocadas no menu** (aula1 usa `#c0…#c19`, capítulos do vídeo com timestamp; os 3 `apostila-pratique` e `aula5/apostila` já eram completos).

**Prova por comportamento (clique real, 1366×768):** todos os itens de menu das 11 apostilas foram clicados e o alvo conferido com `elementFromPoint` — **150 de 150 cliques alcançam o alvo visível, 0 mortos, 0 cobertos** (antes, no arquivo auditado, os 75 links testados alcançavam — o defeito era a *ausência* de entradas, agora medida como contagem).

| apostila | entradas = páginas? | cliques | alcançam o alvo |
|---|---|---|---|
| aula2 / aula3 / aula4 / aula5-51 / aula5-5.2 / aula6 | **sim (27/13/14/15/12/26)** | 107 | 107 |
| aula1, aula5/apostila, 3 apostila-pratique | sim (inalteradas) | 43 | 43 |
| **total** | — | **150** | **150** |

> Nota de precisão (aula 1): são **21 entradas para 20 capítulos** (`#c0…#c19`) porque `#c0` aparece **duas vezes** — o atalho "▸ Ver a aula inteira" e a capa. O índice já era completo (era a aula sem página faltando) e **não foi tocado**. As demais: aula5/apostila 10/10, pratique 4/4, 5/5 e 3/3.

### F.2 D2 — menu lateral no celular → colapsável, fechado por padrão

**Antes (390×844):** o menu era um bloco estático **antes do título** — 1401 px na aula 1, 829 px na aula 6, 674 px na aula 2, 501/465 px na aula 5, 319 px na aula 3, 283 px na aula 4, 210–283 px nas pratique — e o título da aula só aparecia a **914 px** (aula 6) / **1547 px** (aula 1) do topo do documento.
**Depois (390×844):** o menu abre **recolhido** (`display:none`, 0 links visíveis) e o título fica a **155 px** (aula 1: 217 px) — **na primeira tela** de um viewport de 844 px.

| apostila (390×844) | menu antes | menu depois | título antes (px do topo) | título depois |
|---|---|---|---|---|
| aula1-fundamentos | 1401 px | **0 (recolhido)** | 1547 | **217** |
| aula6-equilibrio-quimico | 829 px | **0** | 914 | **155** |
| aula2-gibbs-helmholtz | 674 px | **0** | 759 | **155** |
| aula5-misturas/apostila-sistemas-binarios | 501 px | **0** | 586 | **155** |
| aula5-misturas/apostila | 465 px | **0** | 550 | **155** |
| aula5-misturas/apostila-51 | 449 px | **0** | 534 | **155** |
| aula3 / aula4 / pratique (5 arquivos) | 210–319 px | **0** | 295–404 | **155** |

**Prova por comportamento — botão, teclado e foco (as 11 apostilas, Chromium 390×844):**

| ação | resultado medido |
|---|---|
| estado inicial | `nav` `display:none`/**0 links visíveis**, `aria-expanded="false"`, rótulo "☰ Índice da aula (N)" |
| 1 × `Tab` a partir do `body` | foco no botão (`nav-toggle`) em **todas** as 11 |
| `Enter` no botão | menu abre: **N links visíveis = N páginas de deck** (27/26/15/14/13/12/10/5/4/3/21), `aria-expanded="true"` |
| clique no botão | fecha (`aria-expanded="false"`) e **devolve o foco ao botão** |
| `Escape` | fecha e devolve o foco ao botão |
| clique num item do menu | fecha o menu (o alvo já foi para a tela) |

**Desktop não mudou** (1366×768, medida antes × depois, idênticas): barra lateral **300 px** (290 px na aula 1), `position:sticky`, altura 768 px, título a 64 px (aula 1: 95 px), **sem rolagem horizontal** (`scrollWidth == 1366`) e o botão invisível. O recolhimento só vale abaixo do breakpoint **que cada apostila já tinha** (820 px na aula 1; 900 px nas demais) — ou seja, exatamente onde o menu já deixava de ser coluna e virava bloco empilhado; a faixa pedida (< ~768 px) está inteiramente coberta. **Sem JavaScript** o menu continua como antes (bloco), porque o botão é criado em runtime.

**Verificado também na URL no ar** (abaixo, F.4).

### F.3 D3 — links do material nas 6 descrições do YouTube

**Antes:** as 6 descrições tinham **0 links** (nenhuma ocorrência de `http`), só o aviso beta + o parágrafo de apresentação.
**Depois:** cada uma termina com **3 links**, abaixo do aviso beta (que ficou intacto e continua no topo):

```
---

Material desta aula — apostila navegável, slides em PDF e quiz: https://pcmlab-uepb.github.io/fq2-materiais/<slug-da-aula>/
Esta videoaula no YouTube: https://youtu.be/<id>
Todas as aulas de Físico-Química II: https://pcmlab-uepb.github.io/fq2-materiais/
```

| aula | vídeo | slug do material | aviso beta no topo (1×) | material | vídeo | hub | no fim | privacidade |
|---|---|---|---|---|---|---|---|---|
| 1 | S_HuP3SU4oU | aula1-fundamentos | ✔ | ✔ | ✔ | ✔ | ✔ | unlisted |
| 2 | 3Lu2YnmUb5Q | aula2-gibbs-helmholtz | ✔ | ✔ | ✔ | ✔ | ✔ | unlisted |
| 3 | Rv1y02xJD54 | aula3-equilibrio-fases | ✔ | ✔ | ✔ | ✔ | ✔ | unlisted |
| 4 | 8K5DyAdvjjo | aula4-transicoes-fase | ✔ | ✔ | ✔ | ✔ | ✔ | unlisted |
| 5 | HYLnLgFkZf4 | aula5-misturas | ✔ | ✔ | ✔ | ✔ | ✔ | unlisted |
| 6 | g_JhgoxlT_U | aula6-equilibrio-quimico | ✔ | ✔ | ✔ | ✔ | ✔ | unlisted |

**Prova:** leitura independente pela API (`verify_d3.py`, não a auto-avaliação do comando que escreveu) — título **igual** ao de antes nos 6, `unlisted` mantido, aviso beta aparecendo **exatamente 1×** e no **início**, parágrafo de apresentação preservado, os 3 links presentes, e a linha do hub é a **última** das 6 descrições. As 3 URLs de cada descrição (18 no total) respondem **200**.
**Ferramenta:** `tools/youtube_video.py` só sabia *prepend* (`--prepend-arquivo`), o que poria os links **acima** do aviso beta; acrescentei **`--append-arquivo`** (etapa de sandbox `20261018-142957-b6ed3cb4fa2b`; backup em `cache/scratch/audit/backup/youtube_video.py.antes`), com asserções novas (`termina com o texto novo`, `COMECA com o aviso beta`) e o aviso beta usado como **assert** para provar que ele não saiu do topo. O modo antigo (prepend) ficou idêntico.
**Não alterado:** título (6, conferidos um a um), privacidade (**unlisted** segue), aviso beta, parágrafo de apresentação.

### F.4 Publicação e verificação no ar

- **36 URLs** do site (hub, 6 `index.html`, 6 `slides.pdf`, os 4 PDFs extras, 11 apostilas, 6 `quiz.html`, `site.css`, `robots.txt`) → **todas HTTP 200** com `content-type` coerente (`audit/http_live_d1d2.json`).
- O que está **no ar** confere com o repositório: cada apostila servida traz `entradas == páginas` (27/13/14/15/12/26 nas seis corrigidas) e o CSS/JS do menu colapsável (`audit/producao_d1.json`).
- **Comportamento medido contra a URL no ar** (`audit/producao_d2.json`): em 390×844 o menu abre recolhido (`navVis=0/26`, `navH=0`) com o título a 155 px, **abre** no clique (`navVis=26`, `aria-expanded=true`) e **fecha** no clique seguinte; em 1366×768 a barra continua **300 px `sticky`** com os 26 links visíveis e o botão invisível.

### F.5 O que ficou pendente e por quê

| # | o que | motivo |
|---|---|---|
| D1-residual | em **2 páginas da aula 4** e em várias páginas de exercício o **`<h2>` do próprio material é genérico ou truncado** ("Aspectos", "Aspectos termodinâmicos das", "Página 6", "Página 21", "Mostre que") — e é **isso** que o menu agora repete como rótulo | regra do pedido: *não inventar título*. Melhorar esses rótulos é editar o **material** (o `h2` do deck), não o código — fora do escopo desta rodada |
| D2-residual | no celular, a linha de instrução do topo de `<main>` continua dizendo "Navegue pelas seções **à esquerda**" | é **texto do material**; o pedido manda não alterar texto, e a correção seria trocar a frase por "no índice acima" |
| D2-limite | abaixo de 900 px (820 na aula 1) o menu recolhe; entre 901 px e 1024 px ele continua coluna lateral de 300 px | é o comportamento de desktop atual, que o pedido manda **não** quebrar |
| D3-limite | a descrição pública de cada vídeo agora tem 3 links | decisão editorial do professor; os vídeos continuam **unlisted** (não mudei) |
| D4, D5 (de §D) | 5 decks extras só alcançáveis pelo índice da aula; apostila da aula 1 com 9,4 MB | cosméticos, sem relação com D1–D3 — seguem em aberto |
| método | **não houve inspeção visual** em nenhum dos três | o agente não vê imagem; toda conclusão é de geometria medida, clique real e leitura de API. Há capturas/JSON brutos para conferência humana |

**Arquivos de prova desta rodada** (`~/.hermes/profiles/pesquisador-academico-em-quimica/cache/scratch/audit/`):

| arquivo | conteúdo |
|---|---|
| `gen_ops.py` → `ops_d1d2.json` | as 28 operações (D1+D2) propostas e validadas pelo sandbox |
| `d2_probe.py`, `d2_antes.jsonl`, `d2_depois.jsonl` | geometria e toggle antes × depois (390×844 e 1366×768) |
| `d1_menu.jsonl` | clique real nos 150 itens de menu das 11 apostilas |
| `d3/*.txt`, `verify_d3.py` → `d3_verificado.json` | texto acrescentado e leitura independente das 6 descrições |
| `live_probe.py`, `http_live_d1d2.json`, `producao_d1.json`, `producao_d2.json` | as mesmas medidas contra a URL no ar, depois do push |
| `ops_youtube_tool.json`, `backup/youtube_video.py.antes` | a mudança em `tools/youtube_video.py` e o original |
