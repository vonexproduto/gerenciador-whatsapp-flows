# BDD — Cenários de Comportamento
## Gerenciador de WhatsApp Flows — Vonex.AI
> Formato Gherkin · Linguagem de negócio · Foco em comportamento de API/backend
> Referência: CCM-2743

---

## Feature: Listagem de Flows

```gherkin
Feature: Listagem de Flows
  Como usuário do sistema
  Quero visualizar todos os flows cadastrados
  Para ter uma visão geral dos formulários disponíveis no WhatsApp

  Background:
    Dado que existem flows cadastrados no sistema

  Scenario: Listar todos os flows sem filtros
    Dado que não há nenhum filtro ativo
    Quando solicito a listagem de flows
    Então recebo uma lista com todos os flows cadastrados
    E cada flow retorna: id, name, number, status, createdAt e quantidade de campos
    E os flows são ordenados do mais recente para o mais antigo por padrão

  Scenario: Listagem retorna dados corretos de cada status
    Dado que existem flows com status "publicado", "rascunho", "depreciado" e "erro"
    Quando solicito a listagem de flows
    Então cada flow possui o campo status com um dos valores permitidos
    E flows com status "publicado" exibem as ações: visualizar, depreciar e duplicar
    E flows com status "rascunho" exibem as ações: editar, duplicar e excluir
    E flows com status "depreciado" não exibem nenhuma ação disponível
    E flows com status "erro" exibem as ações: editar, duplicar e excluir

  Scenario: Listagem vazia quando não há flows cadastrados
    Dado que não existe nenhum flow cadastrado
    Quando solicito a listagem de flows
    Então recebo uma lista vazia
    E a resposta indica que há 0 itens
```

---

## Feature: Busca e Filtragem de Flows

```gherkin
Feature: Busca e Filtragem de Flows
  Como usuário do sistema
  Quero buscar e filtrar flows por diferentes critérios
  Para encontrar rapidamente o flow desejado

  Scenario: Busca por nome com correspondência parcial
    Dado que existem flows com nome "Coleta de dados Syscob" e "Formulário de negociação"
    Quando realizo uma busca pelo termo "coleta"
    Então recebo apenas o flow "Coleta de dados Syscob"
    E o flow "Formulário de negociação" não aparece no resultado

  Scenario: Busca por nome sem correspondência
    Dado que nenhum flow possui o nome contendo "xyz"
    Quando realizo uma busca pelo termo "xyz"
    Então recebo uma lista vazia
    E a resposta indica que nenhum flow foi encontrado

  Scenario: Filtro por número de WhatsApp
    Dado que existem flows vinculados ao número "11 3777-1887" e ao número "11 3777-9335"
    Quando filtro por número "11 3777-1887"
    Então recebo apenas os flows vinculados ao número "11 3777-1887"
    E nenhum flow do número "11 3777-9335" aparece no resultado

  Scenario: Filtro por status "publicado"
    Dado que existem flows com diferentes status
    Quando filtro por status "publicado"
    Então recebo apenas flows com status "publicado"

  Scenario: Filtro por status "rascunho"
    Dado que existem flows com diferentes status
    Quando filtro por status "rascunho"
    Então recebo apenas flows com status "rascunho"

  Scenario: Filtro por status "depreciado"
    Dado que existem flows com diferentes status
    Quando filtro por status "depreciado"
    Então recebo apenas flows com status "depreciado"

  Scenario: Filtro por status "erro"
    Dado que existem flows com diferentes status
    Quando filtro por status "erro"
    Então recebo apenas flows com status "erro"

  Scenario: Combinação de busca por nome e filtro por número
    Dado que existem múltiplos flows
    Quando realizo busca pelo nome "Coleta" e filtro pelo número "11 3777-1887"
    Então recebo apenas flows cujo nome contém "Coleta" E estão vinculados ao número "11 3777-1887"

  Scenario: Combinação de filtro por número e por status
    Dado que existem múltiplos flows
    Quando filtro pelo número "11 3777-9335" e pelo status "publicado"
    Então recebo apenas flows do número "11 3777-9335" com status "publicado"
```

---

## Feature: Paginação e Ordenação

```gherkin
Feature: Paginação e Ordenação
  Como usuário do sistema
  Quero controlar a paginação e a ordenação dos flows
  Para navegar pelos resultados de forma eficiente

  Scenario: Paginação com 10 itens por página (padrão)
    Dado que existem 25 flows cadastrados
    Quando solicito a listagem na primeira página com 10 itens por página
    Então recebo 10 flows
    E a resposta indica "1–10 de 25 itens"

  Scenario: Paginação com 25 itens por página
    Dado que existem 30 flows cadastrados
    Quando solicito a listagem com 25 itens por página
    Então recebo 25 flows na primeira página
    E a resposta indica "1–25 de 30 itens"

  Scenario: Paginação com 50 itens por página
    Dado que existem 60 flows cadastrados
    Quando solicito a listagem com 50 itens por página
    Então recebo 50 flows na primeira página

  Scenario: Navegar para a segunda página
    Dado que existem 25 flows cadastrados
    E estou na primeira página com 10 itens por página
    Quando navego para a segunda página
    Então recebo os próximos 10 flows
    E a resposta indica "11–20 de 25 itens"

  Scenario: Última página com menos itens que o limite
    Dado que existem 25 flows cadastrados
    Quando solicito a terceira página com 10 itens por página
    Então recebo 5 flows
    E a resposta indica "21–25 de 25 itens"

  Scenario: Ordenação por mais recentes (padrão)
    Dado que existem flows com datas de criação diferentes
    Quando solicito a listagem com ordenação "mais recentes"
    Então o primeiro flow retornado é o criado mais recentemente

  Scenario: Ordenação por mais antigos
    Dado que existem flows com datas de criação diferentes
    Quando solicito a listagem com ordenação "mais antigos"
    Então o primeiro flow retornado é o criado há mais tempo

  Scenario: Ordenação por nome A–Z
    Dado que existem flows com diferentes nomes
    Quando solicito a listagem com ordenação "nome A–Z"
    Então os flows são retornados em ordem alfabética crescente pelo campo name

  Scenario: Paginação reseta ao alterar filtro
    Dado que estou na segunda página da listagem
    Quando aplico um filtro por status "publicado"
    Então a paginação retorna para a primeira página
    E a contagem de itens reflete apenas os flows publicados
```

---

## Feature: Criação de Flow

```gherkin
Feature: Criação de Flow
  Como usuário do sistema
  Quero criar um novo flow
  Para disponibilizar um formulário interativo no WhatsApp

  Scenario: Criar flow com todos os campos obrigatórios e mínimo de 1 campo
    Dado que forneço os dados:
      | name          | "Pesquisa de satisfação"      |
      | category      | "Questionário"                |
      | number        | "11 3777-1887"                |
      | title         | "Como foi seu atendimento?"   |
      | submitButtonText | "Enviar resposta"            |
    E adiciono 1 campo do tipo "short" com label "Avaliação"
    Quando submeto a criação do flow
    Então o flow é criado com status "rascunho"
    E o flow retorna id, name, category, number, title, submitButtonText, status e createdAt
    E a resposta tem status HTTP 201

  Scenario: Categoria é obrigatória e aceita apenas valores do enum
    Dado que forneço os dados obrigatórios do flow
    E informo category com um dos valores: "Cadastro", "Login", "Agendamento", "Geração de Leads", "Contate-me", "Suporte ao cliente", "Questionário" ou "Outros"
    Quando submeto a criação
    Então o flow é criado com a categoria informada
    E a resposta tem status HTTP 201

  Scenario: Falha ao criar flow sem o campo "category"
    Dado que não informo a categoria do flow
    Quando submeto a criação do flow
    Então recebo status HTTP 422
    E a resposta indica que o campo "category" é obrigatório

  Scenario: Falha ao criar flow com categoria fora do enum
    Dado que informo category com valor "Categoria Inexistente"
    Quando submeto a criação do flow
    Então recebo status HTTP 422
    E a resposta indica que a categoria informada é inválida

  Scenario: Criar flow com mensagem de confirmação opcional
    Dado que forneço os dados obrigatórios do flow
    E informo confirmationMessage como "Obrigado pelo seu contato!"
    Quando submeto a criação
    Então o flow é criado com o campo confirmationMessage preenchido
    E a resposta tem status HTTP 201

  Scenario: Criar flow sem mensagem de confirmação
    Dado que forneço os dados obrigatórios do flow
    E não informo o campo confirmationMessage
    Quando submeto a criação
    Então o flow é criado com confirmationMessage vazio ou nulo
    E a resposta tem status HTTP 201

  Scenario: Falha ao criar flow sem o campo "name"
    Dado que forneço todos os dados obrigatórios exceto o campo "name"
    Quando submeto a criação do flow
    Então recebo status HTTP 422
    E a resposta indica que o campo "name" é obrigatório

  Scenario: Falha ao criar flow sem o campo "number"
    Dado que não informo o número do WhatsApp
    Quando submeto a criação do flow
    Então recebo status HTTP 422
    E a resposta indica que o campo "number" é obrigatório

  Scenario: Falha ao criar flow sem o campo "title"
    Dado que não informo o título da tela
    Quando submeto a criação do flow
    Então recebo status HTTP 422
    E a resposta indica que o campo "title" é obrigatório

  Scenario: Falha ao criar flow sem o campo "submitButtonText"
    Dado que não informo o texto do botão de envio
    Quando submeto a criação do flow
    Então recebo status HTTP 422
    E a resposta indica que o campo "submitButtonText" é obrigatório

  Scenario: Flow novo sempre inicia como rascunho
    Dado que forneço todos os dados válidos para criação
    Quando submeto a criação do flow
    Então o flow é criado com status "rascunho"
    E não é possível criar um flow diretamente com status "publicado"
```

---

## Feature: Edição de Flow

```gherkin
Feature: Edição de Flow
  Como usuário do sistema
  Quero editar um flow existente
  Para corrigir ou atualizar suas informações

  Scenario: Editar flow em status "rascunho"
    Dado que existe um flow com status "rascunho"
    Quando altero o nome do flow para "Novo nome"
    E submeto a atualização
    Então o flow é atualizado com o novo nome
    E permanece com status "rascunho"
    E a resposta tem status HTTP 200

  Scenario: Editar flow em status "erro"
    Dado que existe um flow com status "erro"
    Quando corrijo os dados e submeto a atualização
    Então o flow é atualizado
    E permanece com status "erro" até ser re-publicado
    E a resposta tem status HTTP 200

  Scenario: Bloquear edição de flow "publicado"
    Dado que existe um flow com status "publicado"
    Quando tento alterar qualquer campo do flow
    Então recebo status HTTP 409
    E a resposta indica que flows publicados não podem ser editados

  Scenario: Bloquear edição de flow "depreciado"
    Dado que existe um flow com status "depreciado"
    Quando tento alterar qualquer campo do flow
    Então recebo status HTTP 409
    E a resposta indica que flows depreciados não podem ser editados

  Scenario: Editar flow inexistente
    Dado que não existe flow com id "999"
    Quando tento editar o flow de id "999"
    Então recebo status HTTP 404
    E a resposta indica que o flow não foi encontrado
```

---

## Feature: Gerenciamento de Campos

```gherkin
Feature: Gerenciamento de Campos
  Como usuário do sistema
  Quero gerenciar os campos de um flow
  Para compor o formulário que será exibido no WhatsApp

  # Tipos de campo disponíveis (MVP):
  #   short    → Resposta curta (TextInput)
  #   text     → Resposta longa (TextArea)
  #   radio    → Seleção única (RadioButtonsGroup)
  #   checkbox → Múltipla escolha (CheckboxGroup)
  #   dropdown → Dropdown
  #   date     → Data (DatePicker)
  #   consent  → Campo de consentimento (OptIn)

  Scenario: Adicionar campo do tipo "short" (Resposta curta) a um flow
    Dado que existe um flow em modo de edição
    Quando adiciono um campo com type "short" e label "Nome completo"
    Então o campo é criado com required=true por padrão
    E o campo é criado com inputType "text" por padrão
    E o campo é adicionado ao final da lista de campos do flow

  Scenario: Definir o subtipo de um campo "short"
    Dado que existe um campo do tipo "short"
    Quando defino o inputType como um dos valores: "text", "email", "number", "phone", "password" ou "passcode"
    Então o campo é atualizado com o inputType informado

  Scenario: Falha ao definir inputType fora do enum em campo "short"
    Dado que existe um campo do tipo "short"
    Quando defino o inputType como "cpf"
    Então recebo status HTTP 422
    E a resposta indica que o inputType informado é inválido

  Scenario: Adicionar campo do tipo "text" (Resposta longa) a um flow
    Dado que existe um flow em modo de edição
    Quando adiciono um campo com type "text" e label "Comentários"
    Então o campo é criado como texto livre (multi-linha), sem subtipo e sem opções

  Scenario: Adicionar campo do tipo "consent" (Campo de consentimento)
    Dado que existe um flow em modo de edição
    Quando adiciono um campo com type "consent" e label "Aceito os termos de uso"
    Então o campo é criado sem opções
    E o label (descrição do consentimento) aceita até 300 caracteres

  Scenario: Adicionar instruções opcionais a campos short, text e date
    Dado que existe um campo do tipo "short", "text" ou "date"
    Quando informo o campo opcional "instruction" com até 80 caracteres
    Então o campo é salvo com a instrução
    E a instrução é exibida como texto de apoio abaixo do campo

  Scenario: Instrução não é permitida em tipos que não a suportam
    Dado que existe um campo do tipo "consent"
    Então o campo não expõe o atributo opcional "instruction"

  Scenario: Adicionar campo do tipo "radio" com 2 opções
    Dado que existe um flow em edição
    Quando adiciono um campo com type "radio", label "Motivo" e opções ["Negociação", "Dúvida"]
    Então o campo é criado com as opções informadas
    E a resposta tem status HTTP 201

  Scenario: Adicionar campo do tipo "checkbox" com 2 opções
    Dado que existe um flow em edição
    Quando adiciono um campo com type "checkbox", label "Canais" e opções ["WhatsApp", "E-mail"]
    Então o campo é criado corretamente

  Scenario: Adicionar campo do tipo "dropdown" com 2 opções
    Dado que existe um flow em edição
    Quando adiciono um campo com type "dropdown", label "Departamento" e opções ["Financeiro", "Comercial"]
    Então o campo é criado corretamente

  Scenario: Adicionar campo do tipo "date"
    Dado que existe um flow em edição
    Quando adiciono um campo com type "date" e label "Data de nascimento"
    Então o campo é criado sem necessidade de opções

  Scenario: Falha ao criar campo radio/checkbox/dropdown com menos de 2 opções
    Dado que existe um flow em edição
    Quando adiciono um campo com type "radio" e apenas 1 opção preenchida
    Então recebo status HTTP 422
    E a resposta indica que este tipo de campo exige ao menos 2 opções

  Scenario: Campo "radio" (Seleção única) limita a 5 opções
    Dado que existe um campo do tipo "radio" com 5 opções
    Quando tento adicionar uma 6ª opção
    Então a operação é bloqueada
    E a resposta indica o máximo de 5 opções em Seleção única
    E o limite de 5 opções não se aplica a "checkbox" nem a "dropdown"

  Scenario: Falha ao criar campo sem label
    Dado que existe um flow em edição
    Quando adiciono um campo sem informar o label
    Então recebo status HTTP 422
    E a resposta indica que o label do campo é obrigatório

  Scenario: Reordenar campos — mover campo para cima
    Dado que um flow possui campos na ordem [Campo A, Campo B, Campo C]
    Quando solicito mover o "Campo B" para cima (posição anterior)
    Então a ordem passa a ser [Campo B, Campo A, Campo C]

  Scenario: Reordenar campos — mover campo para baixo
    Dado que um flow possui campos na ordem [Campo A, Campo B, Campo C]
    Quando solicito mover o "Campo B" para baixo (próxima posição)
    Então a ordem passa a ser [Campo A, Campo C, Campo B]

  Scenario: Não mover campo já na primeira posição para cima
    Dado que um flow possui campos na ordem [Campo A, Campo B]
    Quando solicito mover o "Campo A" para cima
    Então a ordem permanece inalterada [Campo A, Campo B]

  Scenario: Não mover campo já na última posição para baixo
    Dado que um flow possui campos na ordem [Campo A, Campo B]
    Quando solicito mover o "Campo B" para baixo
    Então a ordem permanece inalterada [Campo A, Campo B]

  Scenario: Reordenar campos por arrastar e soltar (drag-and-drop)
    Dado que um flow possui campos na ordem [Campo A, Campo B, Campo C]
    Quando seguro o campo "Campo C" pela alça e solto sobre a posição do "Campo A"
    Então a ordem é atualizada refletindo a nova posição
    E a nova ordem (order) é persistida via autosave

  Scenario: Remover campo de um flow
    Dado que um flow possui 3 campos
    Quando removo o segundo campo
    Então o flow passa a ter 2 campos
    E o campo removido não existe mais no flow

  Scenario: Alterar campo de obrigatório para opcional
    Dado que um campo foi criado com required=true
    Quando altero o campo para required=false
    Então o campo é atualizado com required=false

  Scenario: Remover opção de campo com 3 opções
    Dado que um campo do tipo "radio" possui 3 opções
    Quando removo uma opção
    Então o campo passa a ter 2 opções
    E a operação é permitida

  Scenario: Falha ao remover opção quando campo possui apenas 2 opções
    Dado que um campo do tipo "radio" possui apenas 2 opções
    Quando tento remover uma opção
    Então recebo erro indicando que o mínimo de 2 opções deve ser respeitado
    E a opção não é removida
```

---

## Feature: Limites de Caracteres dos Campos

```gherkin
Feature: Limites de Caracteres dos Campos
  Como sistema
  Quero aplicar os limites de caracteres definidos pela Meta a cada campo
  Para garantir que os flows publicados respeitem as regras dos componentes

  # Limites de label por tipo:
  #   short / text / date          → 20 caracteres
  #   radio / checkbox / dropdown  → 30 caracteres
  #   consent (descrição)          → 300 caracteres
  # Título de opção (radio/checkbox/dropdown) → 30 caracteres
  # Texto do botão de envio (submitButtonText) → 30 caracteres
  # Instrução opcional (short/text/date)       → 80 caracteres

  Scenario: Label de short/text/date limitado a 20 caracteres
    Dado que existe um campo do tipo "short", "text" ou "date"
    Quando informo um label com mais de 20 caracteres
    Então recebo status HTTP 422
    E a resposta indica que o label excede o limite de 20 caracteres

  Scenario: Label de radio/checkbox/dropdown limitado a 30 caracteres
    Dado que existe um campo do tipo "radio", "checkbox" ou "dropdown"
    Quando informo um label com mais de 30 caracteres
    Então recebo status HTTP 422
    E a resposta indica que o label excede o limite de 30 caracteres

  Scenario: Título de opção limitado a 30 caracteres
    Dado que existe um campo do tipo "radio", "checkbox" ou "dropdown"
    Quando informo o título de uma opção com mais de 30 caracteres
    Então recebo status HTTP 422
    E a resposta indica que a opção excede o limite de 30 caracteres

  Scenario: Descrição do campo de consentimento limitada a 300 caracteres
    Dado que existe um campo do tipo "consent"
    Quando informo um label (descrição) com mais de 300 caracteres
    Então recebo status HTTP 422
    E a resposta indica que a descrição excede o limite de 300 caracteres

  Scenario: Texto do botão de envio limitado a 30 caracteres
    Dado que estou editando um flow
    Quando informo submitButtonText com mais de 30 caracteres
    Então recebo status HTTP 422
    E a resposta indica que o texto do botão excede o limite de 30 caracteres

  Scenario: Instrução opcional limitada a 80 caracteres
    Dado que existe um campo do tipo "short", "text" ou "date"
    Quando informo instruction com mais de 80 caracteres
    Então recebo status HTTP 422
    E a resposta indica que a instrução excede o limite de 80 caracteres
```

---

## Feature: Validação para Publicação

```gherkin
Feature: Validação para Publicação
  Como sistema
  Quero validar todos os campos obrigatórios antes de permitir a publicação
  Para garantir que apenas flows completos e válidos sejam publicados

  Scenario: Validação bem-sucedida com todos os dados corretos
    Dado que um flow possui:
      | name             | preenchido          |
      | number           | selecionado         |
      | title            | preenchido          |
      | submitButtonText | preenchido          |
      | campos           | ao menos 1          |
    E todos os campos possuem label
    E campos radio/checkbox/dropdown possuem ao menos 2 opções cada
    Quando solicito a validação para publicação
    Então a validação retorna sucesso
    E nenhum erro é indicado

  Scenario: Falha de validação — name vazio
    Dado que o flow não possui nome preenchido
    Quando solicito publicação
    Então recebo erro: "Nome do flow é obrigatório"
    E o status do flow não é alterado

  Scenario: Falha de validação — categoria não selecionada
    Dado que o flow não possui categoria selecionada
    Quando solicito publicação
    Então recebo erro: "Categoria do flow é obrigatória"

  Scenario: Falha de validação — número não selecionado
    Dado que o flow não possui número do WhatsApp vinculado
    Quando solicito publicação
    Então recebo erro: "Número do WhatsApp é obrigatório"

  Scenario: Falha de validação — title vazio
    Dado que o flow não possui título da tela
    Quando solicito publicação
    Então recebo erro: "Título da tela é obrigatório"

  Scenario: Falha de validação — submitButtonText vazio
    Dado que o texto do botão de envio está vazio
    Quando solicito publicação
    Então recebo erro: "Texto do botão de envio é obrigatório"

  Scenario: Falha de validação — sem campos na tela
    Dado que o flow não possui nenhum campo adicionado
    Quando solicito publicação
    Então recebo erro: "Adicione ao menos um campo à tela antes de publicar o flow"

  Scenario: Falha de validação — campo sem label
    Dado que o flow possui um campo com label vazio
    Quando solicito publicação
    Então recebo erro indicando qual campo está sem label

  Scenario: Falha de validação — campo radio com apenas 1 opção preenchida
    Dado que o flow possui um campo do tipo "radio" com apenas 1 opção preenchida
    Quando solicito publicação
    Então recebo erro indicando que o campo exige ao menos 2 opções

  Scenario: Falha de validação — campo checkbox sem opções preenchidas
    Dado que o flow possui um campo do tipo "checkbox" com todas as opções em branco
    Quando solicito publicação
    Então recebo erro indicando que o campo exige ao menos 2 opções

  Scenario: Exibir resumo correto antes da publicação
    Dado que a validação passa com sucesso
    Quando o sistema exibe o resumo antes de confirmar a publicação
    Então o resumo contém a quantidade de campos do flow
    E o resumo contém o número do WhatsApp vinculado ao flow
```

---

## Feature: Publicação de Flow

```gherkin
Feature: Publicação de Flow
  Como usuário do sistema
  Quero publicar um flow após confirmação
  Para disponibilizá-lo para uso em templates e jornadas

  Scenario: Publicar flow em status "rascunho" com dados válidos
    Dado que existe um flow com status "rascunho" e todos os dados válidos
    Quando confirmo a publicação do flow
    Então o flow tem seu status alterado para "publicado"
    E a resposta tem status HTTP 200
    E o flow não pode mais ser editado

  Scenario: Publicar flow em status "erro" após correção
    Dado que existe um flow com status "erro" que foi corrigido
    Quando confirmo a publicação
    Então o flow tem seu status alterado para "publicado"
    E a resposta tem status HTTP 200

  Scenario: Tentar publicar flow já "publicado"
    Dado que existe um flow com status "publicado"
    Quando solicito a publicação novamente
    Então recebo status HTTP 409
    E a resposta indica que o flow já está publicado

  Scenario: Tentar publicar flow "depreciado"
    Dado que existe um flow com status "depreciado"
    Quando solicito a publicação
    Então recebo status HTTP 409
    E a resposta indica que flows depreciados não podem ser republicados

  Scenario: Publicar flow inexistente
    Dado que não existe flow com id "999"
    Quando solicito a publicação do flow "999"
    Então recebo status HTTP 404

  Scenario: Flow publicado perde a possibilidade de edição
    Dado que um flow acabou de ser publicado
    Quando tento editar qualquer campo do flow
    Então recebo status HTTP 409
    E a resposta indica que o flow está publicado e não pode ser alterado
```

---

## Feature: Erro de Publicação retornado pela Meta

```gherkin
Feature: Erro de Publicação retornado pela Meta
  Como sistema
  Quero tratar os erros retornados pela API da Meta durante a publicação
  Para que o usuário entenda o motivo e possa corrigir o flow

  Scenario: Publicação rejeitada pela Meta após confirmação
    Dado que um flow passou na validação interna
    E confirmei o envio para publicação
    Quando a Meta retorna erro no processamento do flow
    Então a publicação é impedida
    E o status do flow passa a "erro" (exibido como "Erro de publicação")
    E o sistema apresenta o título "Erro de publicação"
    E o sistema apresenta o subtítulo "Não foi possível realizar a publicação. Verifique e corrija os motivos retornados e tente novamente."
    E o sistema lista os "Motivos de erro de publicação" retornados pela Meta

  Scenario: Motivos de erro possíveis retornados pela Meta
    Dado que a publicação foi rejeitada pela Meta
    Quando consulto os motivos retornados
    Então a lista pode conter, entre outros:
      | JSON do Flow inválido ou fora do padrão exigido pela Meta              |
      | Categoria do Flow incompatível com o conteúdo enviado                  |
      | Número de WhatsApp (WABA) não verificado ou sem permissão para Flows   |
      | Rótulo (label) de campo excedendo o limite de caracteres              |
      | Campo obrigatório sem rótulo ou sem opções definidas                  |
      | Já existe um Flow publicado com este nome na conta                     |
      | Instabilidade temporária na API da Meta                               |

  Scenario: Flow com erro pode ser corrigido e republicado
    Dado que um flow está com status "erro" após rejeição da Meta
    Quando corrijo os motivos apontados e solicito publicação novamente
    E a Meta aceita o flow
    Então o status do flow passa para "publicado"
```

---

## Feature: Depreciação de Flow

```gherkin
Feature: Depreciação de Flow
  Como usuário do sistema
  Quero depreciar um flow publicado
  Para desativá-lo sem apagá-lo do histórico

  Scenario: Depreciar flow com status "publicado"
    Dado que existe um flow com status "publicado"
    Quando confirmo a depreciação
    Então o flow tem seu status alterado para "depreciado"
    E a resposta tem status HTTP 200

  Scenario: Exibir aviso de impacto antes de depreciar
    Dado que um flow publicado está sendo usado em 2 templates e 1 jornada
    Quando solicito a depreciação
    Então o sistema informa que 2 template(s) e 1 jornada(s) utilizam o flow
    E exige confirmação explícita antes de prosseguir

  Scenario: Flow depreciado não pode ser reutilizado em novos contextos
    Dado que um flow foi depreciado
    Então o flow está em estado final e não pode ser publicado, editado ou reativado

  Scenario: Tentar depreciar flow em status "rascunho"
    Dado que existe um flow com status "rascunho"
    Quando solicito a depreciação
    Então recebo status HTTP 409
    E a resposta indica que apenas flows publicados podem ser depreciados

  Scenario: Tentar depreciar flow já "depreciado"
    Dado que existe um flow com status "depreciado"
    Quando solicito a depreciação novamente
    Então recebo status HTTP 409
    E a resposta indica que o flow já está depreciado

  Scenario: Tentar depreciar flow com status "erro"
    Dado que existe um flow com status "erro"
    Quando solicito a depreciação
    Então recebo status HTTP 409
    E a resposta indica que apenas flows publicados podem ser depreciados

  Scenario: Depreciar flow inexistente
    Dado que não existe flow com id "999"
    Quando solicito a depreciação do flow "999"
    Então recebo status HTTP 404
```

---

## Feature: Exclusão de Flow

```gherkin
Feature: Exclusão de Flow
  Como usuário do sistema
  Quero excluir permanentemente um flow
  Para remover formulários indesejados ou incorretos

  Scenario: Excluir flow com status "rascunho"
    Dado que existe um flow com status "rascunho"
    Quando confirmo a exclusão
    Então o flow é removido permanentemente do sistema
    E a resposta tem status HTTP 200 ou 204

  Scenario: Excluir flow com status "erro"
    Dado que existe um flow com status "erro"
    Quando confirmo a exclusão
    Então o flow é removido permanentemente
    E a resposta tem status HTTP 200 ou 204

  Scenario: Tentar excluir flow "publicado"
    Dado que existe um flow com status "publicado"
    Quando solicito a exclusão
    Então recebo status HTTP 409
    E a resposta indica que flows publicados não podem ser excluídos

  Scenario: Tentar excluir flow "depreciado"
    Dado que existe um flow com status "depreciado"
    Quando solicito a exclusão
    Então recebo status HTTP 409
    E a resposta indica que flows depreciados não podem ser excluídos

  Scenario: Exclusão requer confirmação explícita
    Dado que existe um flow com status "rascunho"
    Quando solicito a exclusão sem confirmar
    Então a exclusão não é executada
    E o sistema exige confirmação antes de prosseguir

  Scenario: Excluir flow inexistente
    Dado que não existe flow com id "999"
    Quando solicito a exclusão do flow "999"
    Então recebo status HTTP 404

  Scenario: Exclusão remove também os campos e opções associados
    Dado que um flow com status "rascunho" possui 3 campos com opções
    Quando confirmo a exclusão
    Então o flow é removido
    E todos os campos e opções relacionados são removidos em cascata
```

---

## Feature: Duplicação de Flow

```gherkin
Feature: Duplicação de Flow
  Como usuário do sistema
  Quero duplicar um flow existente
  Para reutilizar sua estrutura como base para um novo formulário

  Scenario: Duplicar flow com status "publicado"
    Dado que existe um flow "Coleta de dados" com status "publicado"
    Quando solicito a duplicação
    Então um novo flow é criado com nome "Coleta de dados (cópia)"
    E o novo flow possui status "rascunho"
    E todos os campos e opções do flow original são copiados
    E a resposta tem status HTTP 201

  Scenario: Duplicar flow com status "rascunho"
    Dado que existe um flow com status "rascunho"
    Quando solicito a duplicação
    Então um novo flow é criado com sufixo " (cópia)" no nome
    E o novo flow possui status "rascunho"

  Scenario: Duplicar flow com status "erro"
    Dado que existe um flow com status "erro"
    Quando solicito a duplicação
    Então um novo flow é criado com sufixo " (cópia)" no nome
    E o novo flow possui status "rascunho"

  Scenario: Tentar duplicar flow "depreciado"
    Dado que existe um flow com status "depreciado"
    Quando solicito a duplicação
    Então recebo status HTTP 409
    E a resposta indica que flows depreciados não podem ser duplicados

  Scenario: Flow duplicado é independente do original
    Dado que um flow foi duplicado
    Quando altero dados do flow original
    Então o flow duplicado não é afetado
    E quando altero dados do flow duplicado, o flow original não é afetado

  Scenario: Duplicação inclui todos os campos do tipo com opções
    Dado que o flow original possui um campo "radio" com opções ["Sim", "Não"]
    Quando duplico o flow
    Então o campo "radio" da cópia possui as mesmas opções ["Sim", "Não"]

  Scenario: Duplicar flow inexistente
    Dado que não existe flow com id "999"
    Quando solicito a duplicação do flow "999"
    Então recebo status HTTP 404
```

---

## Feature: Autosave de Rascunho

```gherkin
Feature: Autosave de Rascunho
  Como usuário do sistema
  Quero que minhas alterações sejam salvas automaticamente como rascunho
  Para não perder progresso durante a edição

  Scenario: Autosave disparado ao alterar o nome do flow
    Dado que estou editando um flow em rascunho
    Quando altero o campo "name"
    Então o sistema inicia o processo de autosave
    E após o intervalo de debounce (~900ms) o rascunho é salvo
    E a confirmação de "Rascunho salvo" é indicada

  Scenario: Autosave disparado ao alterar o número do WhatsApp
    Dado que estou editando um flow em rascunho
    Quando altero o número do WhatsApp vinculado
    Então o autosave é acionado

  Scenario: Autosave disparado ao alterar o título da tela
    Dado que estou editando um flow em rascunho
    Quando altero o campo "title"
    Então o autosave é acionado

  Scenario: Autosave disparado ao alterar texto do botão
    Dado que estou editando um flow em rascunho
    Quando altero o campo "submitButtonText"
    Então o autosave é acionado

  Scenario: Autosave disparado ao adicionar um campo
    Dado que estou editando um flow em rascunho
    Quando adiciono um novo campo ao flow
    Então o autosave é acionado

  Scenario: Autosave disparado ao remover um campo
    Dado que estou editando um flow em rascunho
    Quando removo um campo do flow
    Então o autosave é acionado

  Scenario: Autosave disparado ao editar label de um campo
    Dado que estou editando um flow em rascunho com ao menos 1 campo
    Quando altero o label de um campo
    Então o autosave é acionado

  Scenario: Autosave disparado ao alterar opções de campo
    Dado que estou editando um campo do tipo "radio"
    Quando altero o texto de uma das opções
    Então o autosave é acionado

  Scenario: Autosave com debounce — múltiplas edições rápidas resultam em um único save
    Dado que estou digitando no campo "name"
    Quando realizo 5 alterações consecutivas em menos de 900ms
    Então o sistema salva apenas uma vez ao final do intervalo de debounce
    E não são gerados 5 saves separados

  Scenario: Autosave não altera o status do flow para "publicado"
    Dado que um flow está em status "rascunho"
    Quando o autosave é executado
    Então o flow permanece com status "rascunho"

  Scenario: Salvamento manual exibe confirmação "Rascunho salvo"
    Dado que estou editando um flow em rascunho
    Quando clico no botão "Salvar rascunho"
    Então as alterações são persistidas
    E o cabeçalho exibe o indicador "Rascunho salvo"

  Scenario: Indicador "Rascunho salvo" é ocultado ao haver novas edições
    Dado que o indicador "Rascunho salvo" está visível
    Quando realizo uma nova edição em qualquer campo
    Então o indicador "Rascunho salvo" deixa de ser exibido
    E sinaliza que há alterações ainda não salvas
```

---

## Feature: Controle de Acesso por Status

```gherkin
Feature: Controle de Acesso por Status
  Como sistema
  Quero controlar quais ações são permitidas por status de flow
  Para garantir a integridade do ciclo de vida dos formulários

  Scenario: Ações permitidas para flow em "rascunho"
    Dado que existe um flow com status "rascunho"
    Então as seguintes ações estão disponíveis: editar, duplicar, excluir
    E as seguintes ações estão bloqueadas: visualizar (como publicado), depreciar

  Scenario: Ações permitidas para flow "publicado"
    Dado que existe um flow com status "publicado"
    Então as seguintes ações estão disponíveis: visualizar, depreciar, duplicar
    E as seguintes ações estão bloqueadas: editar, excluir

  Scenario: Nenhuma ação disponível para flow "depreciado"
    Dado que existe um flow com status "depreciado"
    Então nenhuma ação está disponível para o flow
    E qualquer tentativa de ação resulta em status HTTP 409

  Scenario: Ações permitidas para flow com status "erro"
    Dado que existe um flow com status "erro"
    Então as seguintes ações estão disponíveis: editar, duplicar, excluir
    E o flow pode ser corrigido e re-publicado

  Scenario: Transição válida — rascunho para publicado
    Dado que existe um flow com status "rascunho" e dados válidos
    Quando publico o flow
    Então o status transita de "rascunho" para "publicado"

  Scenario: Transição válida — publicado para depreciado
    Dado que existe um flow com status "publicado"
    Quando deprecio o flow
    Então o status transita de "publicado" para "depreciado"

  Scenario: Transição válida — erro para publicado
    Dado que existe um flow com status "erro" e os problemas foram corrigidos
    Quando publico o flow
    Então o status transita de "erro" para "publicado"

  Scenario: Estado final — depreciado não tem transição de saída
    Dado que existe um flow com status "depreciado"
    Quando tento alterar seu status para qualquer outro valor
    Então recebo status HTTP 409
    E a resposta confirma que "depreciado" é um estado final sem saída

  Scenario: Fluxo de status de "erro" não permite depreciar
    Dado que existe um flow com status "erro"
    Quando solicito a depreciação diretamente
    Então recebo status HTTP 409
    E a resposta indica que o flow precisa ser publicado antes de ser depreciado
```

---

*Documento gerado em 28/05/2026 · Atualizado em 11/06/2026 — Referência: CCM-2743*
