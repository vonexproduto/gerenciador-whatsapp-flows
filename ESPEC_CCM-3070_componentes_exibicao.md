# Componentes de Exibição no Construtor de Flows: Imagem, Carrossel, Texto Formatado e Aceite. Prt 4

ID: CCM-3070
Épico: WhatsApp
Prioridade: 9 (Urgente)
Categoria: Upsell Base Atual
Versão: v3.0
Tipo: Nova Funcionalidade
Status da Task: Produto
Cliente: DLoa
Sprint: 7ª Sprint
Responsável Produto: Alves camila
Status Produto: ✅ Pronto Produto
Responsável UX/UI: suzany ribeiro
Status UX/UI: 🎨 Fazendo UX
Documentação: Componentes de Exibição no Construtor de Flows: Imagem, Carrossel, Texto Formatado e Aceite. (https://app.notion.com/p/Componentes-de-Exibi-o-no-Construtor-de-Flows-Imagem-Carrossel-Texto-Formatado-e-Aceite-399b6d8204f4806e8ad3c10ca1e46da6?pvs=21)
Criado em: July 10, 2026 10:55 AM
Situação Front: ⚫ Sem Prazo
Situação Back: ⚫ Sem Prazo
Produto: 🟣 Não Iniciado
UX: 🟣 Não Iniciado
Story Points: 0
Criado por: Vonex Integrações
Falta validar: Maria Clara Coimbra,Alves camila,Lucas Rehem,suzany ribeiro,Paulo Lebtag
Dias restantes para validar: Sem prazo
SLA: Sem prioridade

### 1. Contexto

O Gerenciador de WhatsApp Flows (Prt 1) já permite criar, publicar e depreciar Flows na [Vonex.AI](http://Vonex.AI). Hoje o construtor guiado gera o Flow com **uma tela**, em modo **estático** (o Flow é montado e enviado "pronto", sem conversar com o servidor em tempo real), e oferece apenas estes componentes:

| Componente atual | Tipo | O que faz |
| --- | --- | --- |
| Instrução (`TextBody`) | Exibição | Texto simples de orientação na tela |
| Resposta curta / longa | Entrada | Coleta texto do contato |
| Seleção única / múltipla / dropdown | Entrada | Coleta escolhas |
| Data (`DatePicker`) | Entrada | Coleta uma data |

Ou seja: o Flow atual é **só texto e campos**. Não há como incluir nenhum elemento visual ou de formatação.

### 2. Problema

Sem componentes visuais, o Flow fica pobre para vários casos reais: mostrar a foto de um produto antes do formulário, exibir um banner, apresentar termos com formatação (negrito, lista, link) ou pedir um aceite de consentimento. Hoje isso não é possível.

| Necessidade | Componente atual atende? |
| --- | --- |
| Mostrar uma imagem (produto, banner, instrução visual) | Não |
| Mostrar várias imagens em galeria | Não |
| Formatar texto (negrito, itálico, listas, links) | Não — só texto puro |
| Pedir aceite de termos/consentimento | Não |

### 3. Solução proposta

Adicionar ao construtor de Flow um conjunto de **componentes de exibição** e um componente simples de **aceite**, mantendo a arquitetura atual (uma tela, modo estático):

- **Imagem** — exibe uma imagem única na tela.
- **Carrossel de imagens** — exibe várias imagens em galeria deslizável.
- **Textos** — Título, Subtítulo, Legenda e **Texto formatado** (negrito, itálico, listas, links).
- **Aceite** — caixa de marcar de consentimento (ex.: "Concordo com os termos").

### 4. Objetivo

Enriquecer a tela do Flow com imagens, texto formatado e aceite, ampliando os casos de uso sem sair da arquitetura de uma tela e modo estático.

**Fora do escopo:**

- Coleta de foto ou documento enviados **pelo contato** (`PhotoPicker` / `DocumentPicker`) — quebra o modo estático e exige modo dinâmico + tratamento de arquivos; fica para épico próprio.
- Múltiplas telas por Flow.
- Modo dinâmico / endpoint (`data_exchange`).
- Lógica condicional (mostrar componente conforme resposta).
- Link entre telas (`EmbeddedLink`) — depende de múltiplas telas.
- Pré-preenchimento de campos com variáveis da jornada.

> ⚠️ **Não confundir com o carrossel da Prt 2.** Na Prt 2 (decisão 6.4) ficou definido que "Flow não é compatível com carrossel" — mas aquilo é o **carrossel de template** (cards de produto no template). Aqui o carrossel é a **galeria de imagens dentro da tela do Flow** (`ImageCarousel`), que é outra coisa e é permitida.
> 

### 5. Escopo da melhoria

Esta melhoria contempla:

- Componente **Imagem** (única) no builder, com origem por **upload** ou **URL**
- Componente **Carrossel de imagens**
- Componentes de **texto**: Título (`TextHeading`), Subtítulo (`TextSubheading`), Legenda (`TextCaption`) e **Texto formatado** (`RichText`)
- Componente **Aceite** (`OptIn`)
- Armazenamento da imagem enviada, **conversão para base64** e **validação de formato e tamanho** conforme o limite da Meta
- Reordenação e exclusão dos novos componentes no builder
- Prévia do Flow atualizada com os novos componentes

### 6. Decisão de produto — Só componentes de exibição nesta rodada

Nesta entrega o Flow apenas **exibe** conteúdo (e coleta um aceite simples); ele **não coleta arquivo** do contato.

**Motivo:** coletar foto/documento exige o modo dinâmico com endpoint, arquivos criptografados e armazenamento próprio, além de mexer no mapeamento da Prt 3 e no relatório da Prt 4. Manter só exibição preserva a arquitetura estática e destrava valor rápido.

> Fonte: Media Upload Components (Meta) — [https://developers.facebook.com/docs/whatsapp/flows/reference/media_upload/](https://developers.facebook.com/docs/whatsapp/flows/reference/media_upload/)
> 

### 6.2 Decisão de produto — Origem e armazenamento da imagem

O operador pode informar a imagem de **duas formas**: subindo um arquivo ou colando uma URL. Em **ambos os casos**, a plataforma converte a imagem para **base64** antes de publicar — a Meta exige base64 nos componentes de Imagem e Carrossel — e aplica o limite de tamanho da regra 8.3.

**Motivo:** dar flexibilidade ao operador mantendo o armazenamento padronizado e dentro do limite da Meta.

> Fonte: Componentes (Meta) — [https://developers.facebook.com/docs/whatsapp/flows/reference/components](https://developers.facebook.com/docs/whatsapp/flows/reference/components)
> 

### 6.3 Decisão de produto — Versões de Flow JSON necessárias

O **carrossel de imagens** exige Flow JSON **v7.1 ou superior**, e o **texto formatado** exige **v5.1 ou superior**.

**Motivo:** são as versões em que a Meta liberou esses componentes.

`[dependência — dev]` Confirmar qual versão de Flow JSON o back gera hoje. Se for anterior, carrossel e/ou texto formatado não ficam disponíveis até a atualização.

> Fonte: Componentes (Meta) — [https://developers.facebook.com/docs/whatsapp/flows/reference/components](https://developers.facebook.com/docs/whatsapp/flows/reference/components)
> 

### 7. Comportamento esperado na interface

No construtor, os novos componentes entram na lista de componentes da tela, ao lado dos atuais, agrupados como "Exibição".

**Imagem**

| Campo | Obrigatório | Editável? | Descrição |
| --- | --- | --- | --- |
| Origem da imagem | Sim | Sim | Upload de arquivo **ou** URL (convertida para base64) |
| Texto alternativo | Não (recomendado) | Sim | Descrição da imagem para acessibilidade (leitores de tela) |
| Modo de exibição | Não | Sim | Conter ou Preencher |

**Carrossel de imagens**

| Campo | Obrigatório | Editável? | Descrição |
| --- | --- | --- | --- |
| Lista de imagens | Sim | Sim | 2 a 3 imagens, cada uma com imagem + texto alternativo |
| Aspecto | Não | Sim | 4:3 ou 16:9 — vale para o carrossel inteiro |
| Modo de exibição | Não | Sim | Conter ou Preencher — vale para o carrossel inteiro |

**Textos**

| Componente | Descrição |
| --- | --- |
| Título (`TextHeading`) | Título de destaque |
| Subtítulo (`TextSubheading`) | Subtítulo |
| Legenda (`TextCaption`) | Texto pequeno / observação |
| Texto formatado (`RichText`) | Texto rico: negrito, itálico, listas, links e tabelas |

*(O corpo de texto simples já existe como "Instrução".)*

**Aceite**

| Campo | Obrigatório | Editável? | Descrição |
| --- | --- | --- | --- |
| Texto do aceite | Sim | Sim | Ex.: "Concordo com os termos de uso" (até 120 caracteres). Exibido como caixa de marcar `☐` |
| Obrigatório para enviar? | Sim | Sim | Se o contato precisa marcar para concluir o Flow |

**Prévia:** todos os novos componentes aparecem na prévia do Flow durante a criação, na posição em que foram inseridos.

### 8. Regras de negócio

### 8.1 Componentes de exibição não coletam dado

Imagem, Carrossel e os textos (Título, Subtítulo, Legenda, Texto formatado) **não geram variável**, não aparecem no mapeamento da jornada (Prt 3) nem como coluna no relatório (Prt 4). São apenas visuais.

### 8.2 Pelo menos um campo de entrada continua obrigatório

Herdando a regra da Prt 1 (10.4), um Flow não pode ser publicado só com componentes de exibição — ele precisa de ao menos **1 campo de entrada**. O Aceite (8.11) conta como entrada; imagem e texto não.

Momento de validação: ao publicar o Flow.

Mensagem: `Adicione ao menos um campo de entrada à tela antes de publicar o Flow.`

### 8.3 Como a imagem é guardada e por que existe um limite de tamanho

Existem duas formas de entregar uma imagem para o WhatsApp exibir dentro do Flow: **(a)** informar o endereço (link) de onde a imagem está hospedada, ou **(b)** embutir a imagem inteira dentro da própria configuração do Flow — nesse caso a imagem é convertida em um texto muito longo e "colada por dentro" do Flow. A Meta trabalha com a forma **(b)**: a imagem vai embutida em base64.

Como a imagem embutida deixa o Flow **pesado**, a Meta impõe **limites de tamanho**: cada imagem deve ter idealmente até **~300 kb**, o Flow inteiro não deve passar de **~1 Mb**, e cada tela aceita no **máximo 3 imagens**. Se a imagem for grande demais, a publicação **falha**. Para o operador não esbarrar nisso, a Vonex vai **(1)** validar o tamanho da imagem no envio e **(2)** compactar automaticamente ou bloquear com aviso as imagens acima do limite, para a publicação nunca quebrar por causa de uma imagem pesada.

Momentos de validação: ao adicionar/enviar a imagem no builder; ao publicar o Flow.

Mensagem (imagem acima do limite): `Esta imagem é muito grande e pode impedir a publicação do Flow. Reduza o tamanho ou escolha outra imagem.`

> Fonte: Componentes (Meta) — [https://developers.facebook.com/docs/whatsapp/flows/reference/components](https://developers.facebook.com/docs/whatsapp/flows/reference/components) · Flow JSON — [https://developers.facebook.com/docs/whatsapp/flows/reference/flowjson/](https://developers.facebook.com/docs/whatsapp/flows/reference/flowjson/)
> 

### 8.4 Origem da imagem: upload ou URL

O operador pode informar a imagem subindo um arquivo ou colando uma URL. Em ambos os casos a plataforma **converte para base64** antes de publicar (a Meta exige base64 em Imagem e Carrossel). Quando a origem for URL, a plataforma baixa a imagem para converter (ver 8.6 para falha de carregamento).

### 8.5 Formatos de imagem aceitos

São aceitos apenas **JPG/JPEG e PNG**. Outros formatos (SVG, TIFF, WEBP, etc.) são bloqueados no upload.

Momento de validação: ao enviar/colar a imagem.

Mensagem: `Formato não suportado. Envie uma imagem JPG ou PNG.`

> Fonte: Componentes (Meta) — [https://developers.facebook.com/docs/whatsapp/flows/reference/components](https://developers.facebook.com/docs/whatsapp/flows/reference/components)
> 

### 8.6 Imagem informada por URL que não carrega

Ao informar a imagem por URL, se o endereço for inválido, expirar (timeout) ou a imagem não for encontrada, o builder não adiciona a imagem e exibe aviso. (A imagem publicada vai embutida em base64, então **não "quebra" no celular do contato** — o problema só ocorre neste momento de carregamento no builder.)

Momento de validação: ao colar/validar a URL no builder.

Mensagem: `Não foi possível carregar esta imagem.`

### 8.7 Responsividade — o operador não define largura nem altura

A imagem se adapta automaticamente à largura do cliente WhatsApp. O operador define apenas o **modo de exibição** (Conter / Preencher); largura e altura não são expostas.

> Fonte: Componentes (Meta) — [https://developers.facebook.com/docs/whatsapp/flows/reference/components](https://developers.facebook.com/docs/whatsapp/flows/reference/components)
> 

### 8.8 Texto alternativo

Cada imagem pode ter um texto alternativo (descrição lida por leitores de tela). **No carrossel, o texto alternativo de cada imagem é obrigatório**; na imagem única, é recomendado.

### 8.9 Carrossel — quantidade e configuração

O carrossel exibe de **2 a 3 imagens** (a Meta permite no máximo 3). O aspecto (4:3 ou 16:9) e o modo de exibição (Conter/Preencher) valem para o **carrossel inteiro**, não por imagem. Cada imagem tem imagem + texto alternativo — **não há legenda por imagem** (não suportado pela Meta). Limites da Meta: até 2 carrosséis por tela e 3 por Flow.

Momento de validação: ao salvar/publicar.

Mensagem (menos de 2 imagens): `Adicione ao menos duas imagens ao carrossel.`

> Fonte: Componentes (Meta) — [https://developers.facebook.com/docs/whatsapp/flows/reference/components](https://developers.facebook.com/docs/whatsapp/flows/reference/components)
> 

### 8.10 Texto formatado — formatações e versão

O texto formatado aceita negrito, itálico, listas, links e tabelas. Exige Flow JSON **v5.1+** (ver 6.3). Até a v6.2 ele precisa ficar **sozinho na tela** (não divide a tela com outros componentes); a partir da v6.3 pode conviver com o botão de envio. `[dependência — dev]` confirmar a versão em uso.

> Fonte: Componentes (Meta) — [https://developers.facebook.com/docs/whatsapp/flows/reference/components](https://developers.facebook.com/docs/whatsapp/flows/reference/components)
> 

### 8.11 Aceite (checkbox) gera um valor sim/não

O Aceite é uma **caixa de marcar** (`☐`), com o texto do aceite (até 120 caracteres) e a opção de ser obrigatório para concluir o Flow. **Não é um botão.** Diferente dos componentes de exibição, ele devolve um valor (**aceitou / não aceitou**) e pode ser mapeado como variável na jornada. Limite da Meta: até 5 aceites por tela.

`[dependência — Prt 3 e Prt 4]` o mapeamento da Prt 3 precisa reconhecer um novo tipo "aceite (sim/não)", e o relatório da Prt 4 precisa exibi-lo como coluna. Registrar essa dependência nas duas tasks.

> Fonte: Componentes (Meta) — [https://developers.facebook.com/docs/whatsapp/flows/reference/components](https://developers.facebook.com/docs/whatsapp/flows/reference/components)
> 

### 8.12 Reposicionamento de componentes

Os componentes de exibição seguem a mesma ordenação dos componentes atuais do Flow. O operador pode reposicioná-los por **drag-and-drop** ou pelos **controles de mover para cima/baixo**, conforme o padrão já existente do builder.

### 8.13 Exclusão de componente

Ao remover um componente Imagem ou Carrossel, ele sai apenas da **estrutura do Flow**. O arquivo armazenado pode permanecer no repositório interno conforme a política de retenção da plataforma.

### 8.14 Flow publicado permanece imutável

Adicionar ou trocar uma imagem não muda a regra da Prt 1: um Flow **publicado não pode ser editado**. Para alterar a imagem, é preciso depreciar o Flow e criar um novo.

### 9. Exemplo de uso

Um operador cria um Flow de captação de leads para a Syscob. Ele adiciona um **Título** ("Conheça nosso plano"), um **Carrossel** com três fotos do produto (cada uma com texto alternativo), um **Texto formatado** com os benefícios em lista, um campo de **Resposta curta** (nome) e um **Aceite** ("Autorizo o contato comercial", obrigatório). Uma das fotos era muito grande — a plataforma avisa e ele troca por uma menor. Ele publica o Flow, que passa a poder ser vinculado a um template (Prt 2) e disparado na jornada (Prt 3).

### 10. Critérios de aceite

### Interface

- [ ]  Os componentes Imagem, Carrossel, Título, Subtítulo, Legenda, Texto formatado e Aceite aparecem no construtor
- [ ]  A prévia exibe cada novo componente na posição inserida
- [ ]  A imagem pode ser informada por upload e por URL
- [ ]  O operador consegue reposicionar componentes por drag-and-drop ou mover para cima/baixo
- [ ]  O operador consegue remover Imagem e Carrossel
- [ ]  O Aceite permite definir texto e se é obrigatório, exibido como caixa de marcar

### Validações

- [ ]  Imagem acima do limite (~300 kb por imagem / ~1 Mb no Flow) é compactada ou bloqueada com a mensagem definida
- [ ]  Apenas JPG/JPEG e PNG são aceitos; demais formatos são bloqueados com aviso
- [ ]  URL de imagem que não carrega exibe aviso e não adiciona a imagem
- [ ]  Não é possível publicar um Flow só com componentes de exibição (falta campo de entrada)
- [ ]  Carrossel exige de 2 a 3 imagens

### Integração / Dados

- [ ]  Imagem, Carrossel e textos não geram variável nem coluna de relatório
- [ ]  Imagem e carrossel são enviados à Meta em base64
- [ ]  O operador não define largura nem altura da imagem
- [ ]  O Aceite gera um valor sim/não passível de mapeamento
- [ ]  O JSON gerado usa os componentes no formato correto da Meta

### 11. Cenários de teste

### Cenário 1 — Adicionar imagem por upload

**Dado** que o operador está criando um Flow

**Quando** adiciona o componente Imagem e sobe um arquivo JPG dentro do limite

**Então** a imagem aparece na prévia

**E** o Flow pode ser publicado

### Cenário 2 — Adicionar imagem por URL

**Dado** que o operador está criando um Flow

**Quando** adiciona o componente Imagem informando uma URL válida

**Então** a plataforma converte a imagem para base64 e ela aparece na prévia

### Cenário 3 — Imagem acima do limite

**Dado** que o operador adiciona uma imagem muito grande

**Quando** tenta salvar/publicar

**Então** a plataforma compacta ou bloqueia com a mensagem de tamanho

### Cenário 4 — URL de imagem inválida

**Dado** que o operador cola uma URL inválida, com timeout ou imagem inexistente

**Quando** o builder tenta carregar a imagem

**Então** a imagem não é adicionada e exibe `Não foi possível carregar esta imagem.`

### Cenário 5 — Formato não suportado

**Dado** que o operador tenta subir um arquivo SVG, TIFF ou WEBP

**Quando** envia o arquivo

**Então** a ação é bloqueada com `Formato não suportado. Envie uma imagem JPG ou PNG.`

### Cenário 6 — Carrossel com várias imagens

**Dado** que o operador adiciona um Carrossel com três imagens, cada uma com texto alternativo

**Quando** visualiza a prévia

**Então** as três imagens aparecem em galeria deslizável

### Cenário 7 — Texto formatado

**Dado** que o operador adiciona um Texto formatado com negrito e lista

**Quando** visualiza a prévia

**Então** o texto aparece com a formatação aplicada

### Cenário 8 — Reposicionar componente

**Dado** que o Flow tem uma imagem acima de um texto

**Quando** o operador arrasta o texto para cima da imagem

**Então** a ordem é atualizada na estrutura e na prévia

### Cenário 9 — Excluir imagem

**Dado** que o Flow tem um componente Imagem

**Quando** o operador remove o componente

**Então** a imagem sai da estrutura do Flow (o arquivo pode permanecer no repositório conforme retenção)

### Cenário 10 — Publicar só com exibição (bloqueado)

**Dado** que o Flow tem apenas imagem e texto, sem nenhum campo de entrada

**Quando** o operador tenta publicar

**Então** a ação é bloqueada com a mensagem de "adicione ao menos um campo de entrada"

### Cenário 11 — Aceite gera variável

**Dado** que o Flow tem um componente de Aceite

**Quando** o contato conclui o Flow marcando o aceite

**Então** o valor "sim" fica disponível para mapeamento na jornada

### 12. Mensagens de validação

| Situação | Mensagem |
| --- | --- |
| Imagem acima do limite | `Esta imagem é muito grande e pode impedir a publicação do Flow. Reduza o tamanho ou escolha outra imagem.` |
| Formato de imagem não suportado | `Formato não suportado. Envie uma imagem JPG ou PNG.` |
| URL de imagem não carrega | `Não foi possível carregar esta imagem.` |
| Publicar sem campo de entrada | `Adicione ao menos um campo de entrada à tela antes de publicar o Flow.` |
| Carrossel com menos de 2 imagens | `Adicione ao menos duas imagens ao carrossel.` |

### 13. Referências

| Documento | URL |
| --- | --- |
| Flows — Visão geral | [https://developers.facebook.com/docs/whatsapp/flows/](https://developers.facebook.com/docs/whatsapp/flows/) |
| Flow JSON — estrutura de telas e componentes | [https://developers.facebook.com/docs/whatsapp/flows/reference/flowjson/](https://developers.facebook.com/docs/whatsapp/flows/reference/flowjson/) |
| Componentes — referência de todos os componentes de tela (Image, ImageCarousel, RichText, TextHeading/Subheading/Caption, OptIn) | [https://developers.facebook.com/docs/whatsapp/flows/reference/components](https://developers.facebook.com/docs/whatsapp/flows/reference/components) |
| Media Upload Components — PhotoPicker/DocumentPicker (fora do escopo) | [https://developers.facebook.com/docs/whatsapp/flows/reference/media_upload/](https://developers.facebook.com/docs/whatsapp/flows/reference/media_upload/) |
| Flows API — criação e publicação | [https://developers.facebook.com/docs/whatsapp/flows/reference/flowsapi/](https://developers.facebook.com/docs/whatsapp/flows/reference/flowsapi/) |
| Flows — Changelog (versões do Flow JSON) | [https://developers.facebook.com/docs/whatsapp/flows/changelogs/](https://developers.facebook.com/docs/whatsapp/flows/changelogs/) |
| Sending a Flow — envio do template com botão Flow | [https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/sendingaflow/](https://developers.facebook.com/documentation/business-messaging/whatsapp/flows/guides/sendingaflow/) |

### 14. Wireframes de referência

*Referências de baixa fidelidade para alinhar UX, Front-end e QA sobre a disposição dos componentes e o painel lateral. Não são a arte final.*

**Painel "Adicionar componente"**

```
+ Adicionar componente
──────────────
EXIBIÇÃO
🖼  Imagem
🖼🖼 Carrossel
Aa  Texto formatado
H1  Título
H2  Subtítulo
H3  Legenda
──────────────
ENTRADAS
Resposta curta
Resposta longa
Dropdown
Data
☑  Aceite
```

**Tela do Flow (montagem)**

```
────────────────────────────────────
[Título]  Conheça nosso novo plano
────────────────────────────────────
🖼
┌──────────────────────────────┐
│            IMAGEM            │
└──────────────────────────────┘
────────────────────────────────────
Carrossel        ◀───────────▶
┌────┐ ┌────┐ ┌────┐
│img1│ │img2│ │img3│
└────┘ └────┘ └────┘
────────────────────────────────────
Texto formatado
✔ Benefício 1
✔ Benefício 2
✔ Benefício 3
Saiba mais
────────────────────────────────────
Nome
[__________________________]
────────────────────────────────────
☐ Concordo com os termos
────────────────────────────────────
```

**Painel lateral — Imagem**

```
Imagem
Origem
 (o) Upload
 ( ) URL
────────────────────
Arquivo
[ Escolher arquivo ]
────────────────────
Texto alternativo
[__________________]
────────────────────
Modo
 (•) Conter
 ( ) Preencher
────────────────────
[ Excluir componente ]
```

**Painel lateral — Carrossel**

```
Carrossel
Imagem 1
 [ upload ]
 Texto alternativo
 [__________________]
Imagem 2
 [ upload ]
 Texto alternativo
 [__________________]
Imagem 3
 [ upload ]
 Texto alternativo
 [__________________]
+ Adicionar imagem   (máx. 3)
```

**Prévia final no WhatsApp**

```
┌─────────────────────────┐
 Conheça nosso plano
 █████████████████████
 ◀      imagem       ▶
 Benefícios
  • Atendimento 24h
  • Sem fidelidade
  • Cancelamento simples
 Nome
 [_____________]
 ☐ Concordo com os termos
 [ Enviar ]
└─────────────────────────┘
```