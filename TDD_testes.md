# TDD — Suítes de Teste
## Gerenciador de WhatsApp Flows — Vonex.AI
> Padrão: [ ] Nome · Entrada · Esperado
> Referência: CCM-2743

---

## 1. Testes de Criação — `POST /flows`

```
[ ] Criar flow com todos os campos obrigatórios e 1 campo de tela
    Entrada: {
      name: "Pesquisa de satisfação",
      category: "Questionário",
      number: "11 3777-1887",
      title: "Como foi seu atendimento?",
      submitButtonText: "Enviar resposta",
      confirmationMessage: null,
      fields: [{ type: "short", label: "Avaliação", inputType: "text", required: true, options: [] }]
    }
    Esperado: HTTP 201 · { id: <uuid>, status: "rascunho", createdAt: <datetime>, name: "Pesquisa de satisfação", category: "Questionário", ... }

[ ] Criar flow sem o campo "category"
    Entrada: { name: "Flow X", number: "11 3777-1887", title: "Tela", submitButtonText: "Enviar" }
    Esperado: HTTP 422 · { errors: [{ field: "category", message: "Categoria do flow é obrigatória" }] }

[ ] Criar flow com category fora do enum
    Entrada: { name: "Flow X", category: "Inexistente", number: "11 3777-1887", title: "Tela", submitButtonText: "Enviar" }
    Esperado: HTTP 422 · { errors: [{ field: "category", message: "Categoria inválida" }] }
    # enum: Cadastro · Login · Agendamento · Geração de Leads · Contate-me · Suporte ao cliente · Questionário · Outros

[ ] Criar flow com submitButtonText acima de 30 caracteres
    Entrada: { ...dados válidos..., submitButtonText: "Texto de botão muito longo que excede" }
    Esperado: HTTP 422 · { errors: [{ field: "submitButtonText", message: "Máximo de 30 caracteres" }] }

[ ] Criar flow com mensagem de confirmação preenchida
    Entrada: { ...dados obrigatórios..., confirmationMessage: "Obrigado pelo seu contato!" }
    Esperado: HTTP 201 · { confirmationMessage: "Obrigado pelo seu contato!", status: "rascunho" }

[ ] Criar flow sem mensagem de confirmação (campo omitido)
    Entrada: { name, number, title, submitButtonText } (sem confirmationMessage)
    Esperado: HTTP 201 · { confirmationMessage: null ou "" }

[ ] Criar flow sem o campo "name"
    Entrada: { number: "11 3777-1887", title: "Tela", submitButtonText: "Enviar" }
    Esperado: HTTP 422 · { errors: [{ field: "name", message: "Nome do flow é obrigatório" }] }

[ ] Criar flow com "name" vazio ("")
    Entrada: { name: "", number: "11 3777-1887", title: "Tela", submitButtonText: "Enviar" }
    Esperado: HTTP 422 · { errors: [{ field: "name", message: "Nome do flow é obrigatório" }] }

[ ] Criar flow sem o campo "number"
    Entrada: { name: "Flow X", title: "Tela", submitButtonText: "Enviar" }
    Esperado: HTTP 422 · { errors: [{ field: "number", message: "Número do WhatsApp é obrigatório" }] }

[ ] Criar flow sem o campo "title"
    Entrada: { name: "Flow X", number: "11 3777-1887", submitButtonText: "Enviar" }
    Esperado: HTTP 422 · { errors: [{ field: "title", message: "Título da tela é obrigatório" }] }

[ ] Criar flow sem o campo "submitButtonText"
    Entrada: { name: "Flow X", number: "11 3777-1887", title: "Tela" }
    Esperado: HTTP 422 · { errors: [{ field: "submitButtonText", message: "Texto do botão de envio é obrigatório" }] }

[ ] Criar flow com corpo de requisição vazio
    Entrada: {}
    Esperado: HTTP 422 · { errors: [ lista com todos os campos obrigatórios ausentes ] }

[ ] Flow criado via POST sempre inicia como "rascunho"
    Entrada: { ...dados válidos..., status: "publicado" } (tentativa de forçar status)
    Esperado: HTTP 201 · { status: "rascunho" } (status informado é ignorado)

[ ] Criar flow com múltiplos campos de diferentes tipos
    Entrada: { ...dados obrigatórios..., fields: [
      { type: "short", label: "Nome", inputType: "text", required: true, options: [] },
      { type: "radio", label: "Motivo", required: true, options: ["Negociação", "Dúvida"] },
      { type: "date", label: "Data de nasc.", required: false, options: [] },
      { type: "consent", label: "Aceito os termos", required: true, options: [] }
    ]}
    Esperado: HTTP 201 · { fields: [<4 campos com ids gerados e order correto>] }
```

---

## 2. Testes de Leitura — `GET /flows` e `GET /flows/:id`

```
[ ] Listar flows sem parâmetros — retorna todos os flows
    Entrada: GET /flows
    Esperado: HTTP 200 · { data: [<lista de flows>], total: N, page: 1, perPage: 10 }

[ ] Listar flows — cada item contém os campos esperados
    Entrada: GET /flows
    Esperado: HTTP 200 · cada item contém { id, name, number, status, createdAt, fieldCount }

[ ] Ordenação padrão é "mais recentes"
    Entrada: GET /flows
    Esperado: HTTP 200 · primeiro item é o flow com createdAt mais recente

[ ] Buscar flow por ID existente
    Entrada: GET /flows/1
    Esperado: HTTP 200 · { id: 1, name: "Coleta de dados Syscob", number: "11 3777-1887",
      status: "publicado", title: ..., submitButtonText: ..., fields: [...] }

[ ] Buscar flow por ID inexistente
    Entrada: GET /flows/999
    Esperado: HTTP 404 · { error: "Flow não encontrado" }

[ ] Buscar flow — retorna os campos com suas opções
    Entrada: GET /flows/3  (flow com campo dropdown contendo opções)
    Esperado: HTTP 200 · { fields: [{ ..., options: [{ id, value, order }] }] }

[ ] Listar flows — resposta pagina corretamente com total
    Entrada: GET /flows?page=1&perPage=10  (com 25 flows no banco)
    Esperado: HTTP 200 · { data: [<10 itens>], total: 25, page: 1, perPage: 10, totalPages: 3 }

[ ] Buscar flows — campo fields[] não inclui dados sensíveis desnecessários
    Entrada: GET /flows
    Esperado: HTTP 200 · a listagem não retorna o conteúdo completo dos fields (apenas fieldCount)
```

---

## 3. Testes de Atualização — `PATCH /flows/:id`

```
[ ] Atualizar nome de flow em "rascunho"
    Entrada: PATCH /flows/2 · { name: "Formulário atualizado" }
    Esperado: HTTP 200 · { id: 2, name: "Formulário atualizado", status: "rascunho" }

[ ] Atualizar número do WhatsApp de flow em "rascunho"
    Entrada: PATCH /flows/2 · { number: "11 3777-9335" }
    Esperado: HTTP 200 · { number: "11 3777-9335" }

[ ] Atualizar título da tela de flow em "rascunho"
    Entrada: PATCH /flows/2 · { title: "Novo título da tela" }
    Esperado: HTTP 200 · { title: "Novo título da tela" }

[ ] Atualizar texto do botão de envio
    Entrada: PATCH /flows/2 · { submitButtonText: "Confirmar dados" }
    Esperado: HTTP 200 · { submitButtonText: "Confirmar dados" }

[ ] Atualizar mensagem de confirmação
    Entrada: PATCH /flows/2 · { confirmationMessage: "Dados recebidos!" }
    Esperado: HTTP 200 · { confirmationMessage: "Dados recebidos!" }

[ ] Atualizar flow em status "erro" é permitido
    Entrada: PATCH /flows/5 · { name: "Validação CPF v2" }  (flow 5 tem status "erro")
    Esperado: HTTP 200 · { name: "Validação CPF v2", status: "erro" }

[ ] Tentar atualizar flow "publicado" — bloqueado
    Entrada: PATCH /flows/1 · { name: "Tentativa de edição" }  (flow 1 é "publicado")
    Esperado: HTTP 409 · { error: "Flows publicados não podem ser editados" }

[ ] Tentar atualizar flow "depreciado" — bloqueado
    Entrada: PATCH /flows/4 · { name: "Tentativa" }  (flow 4 é "depreciado")
    Esperado: HTTP 409 · { error: "Flows depreciados não podem ser editados" }

[ ] Atualizar flow inexistente
    Entrada: PATCH /flows/999 · { name: "Qualquer coisa" }
    Esperado: HTTP 404 · { error: "Flow não encontrado" }

[ ] Atualizar "name" para vazio — erro de validação
    Entrada: PATCH /flows/2 · { name: "" }
    Esperado: HTTP 422 · { errors: [{ field: "name", message: "Nome do flow é obrigatório" }] }

[ ] Atualização parcial — campos não informados permanecem inalterados
    Entrada: PATCH /flows/2 · { title: "Só o título muda" }
    Esperado: HTTP 200 · { title: "Só o título muda", name: <valor original mantido>, number: <valor original mantido> }
```

---

## 4. Testes de Exclusão — `DELETE /flows/:id`

```
[ ] Excluir flow com status "rascunho"
    Entrada: DELETE /flows/2  (flow 2 é "rascunho")
    Esperado: HTTP 200 ou 204 · flow removido do banco · GET /flows/2 retorna 404

[ ] Excluir flow com status "erro"
    Entrada: DELETE /flows/5  (flow 5 é "erro")
    Esperado: HTTP 200 ou 204 · flow removido

[ ] Exclusão em cascata — campos e opções são removidos junto
    Entrada: DELETE /flows/2  (possui campos com opções)
    Esperado: HTTP 200 ou 204 · GET /flows/2/fields retorna 404 ou lista vazia

[ ] Tentar excluir flow "publicado"
    Entrada: DELETE /flows/1  (flow 1 é "publicado")
    Esperado: HTTP 409 · { error: "Flows publicados não podem ser excluídos" }

[ ] Excluir flow com status "depreciado"
    Entrada: DELETE /flows/4  (flow 4 é "depreciado")
    Esperado: HTTP 200 ou 204 · flow removido do banco · GET /flows/4 retorna 404

[ ] Excluir flow inexistente
    Entrada: DELETE /flows/999
    Esperado: HTTP 404 · { error: "Flow não encontrado" }

[ ] Exclusão é idempotente — segunda chamada para ID já removido retorna 404
    Entrada: DELETE /flows/2 (segunda vez após já excluído)
    Esperado: HTTP 404 · { error: "Flow não encontrado" }
```

---

## 5. Testes de Ação: Publicar — `POST /flows/:id/publish`

```
[ ] Publicar flow "rascunho" com todos os dados válidos
    Entrada: POST /flows/2/publish  (flow com dados completos e ao menos 1 campo válido)
    Esperado: HTTP 200 · { id: 2, status: "publicado" }

[ ] Publicar flow "erro" após correção
    Entrada: POST /flows/5/publish  (flow com status "erro" e dados corrigidos)
    Esperado: HTTP 200 · { id: 5, status: "publicado" }

[ ] Publicar flow com validação falhando — name vazio
    Entrada: POST /flows/2/publish  (flow com name: "")
    Esperado: HTTP 422 · { errors: [{ field: "name", message: "Nome do flow é obrigatório" }] }

[ ] Publicar flow com validação falhando — sem campos
    Entrada: POST /flows/6/publish  (flow sem nenhum campo)
    Esperado: HTTP 422 · { errors: [{ message: "Adicione ao menos um campo à tela antes de publicar" }] }

[ ] Publicar flow com campo radio sem opções suficientes
    Entrada: POST /flows/2/publish  (flow com campo radio contendo apenas 1 opção preenchida)
    Esperado: HTTP 422 · { errors: [{ field: "fields[0].options", message: "Campo exige ao menos 2 opções" }] }

[ ] Publicar flow com campo sem label
    Entrada: POST /flows/2/publish  (flow com campo cujo label está vazio)
    Esperado: HTTP 422 · { errors: [{ message: "Campo 1: label é obrigatório" }] }

[ ] Tentar publicar flow já "publicado"
    Entrada: POST /flows/1/publish
    Esperado: HTTP 409 · { error: "Flow já está publicado" }

[ ] Tentar publicar flow "depreciado"
    Entrada: POST /flows/4/publish
    Esperado: HTTP 409 · { error: "Flows depreciados não podem ser republicados" }

[ ] Tentar publicar flow inexistente
    Entrada: POST /flows/999/publish
    Esperado: HTTP 404 · { error: "Flow não encontrado" }

[ ] Após publicação o flow não pode ser editado
    Entrada: POST /flows/2/publish (sucesso) → PATCH /flows/2 { name: "Edição pós-publicação" }
    Esperado: POST retorna 200 · PATCH retorna 409
```

---

## 6. Testes de Ação: Depreciar — `POST /flows/:id/deprecate`

```
[ ] Depreciar flow "publicado"
    Entrada: POST /flows/1/deprecate
    Esperado: HTTP 200 · { id: 1, status: "depreciado" }

[ ] Depreciar flow "publicado" — retorna aviso de impacto
    Entrada: POST /flows/1/deprecate
    Esperado: HTTP 200 · { status: "depreciado", impact: { templates: N, journeys: M } }

[ ] Tentar depreciar flow "rascunho"
    Entrada: POST /flows/2/deprecate
    Esperado: HTTP 409 · { error: "Apenas flows publicados podem ser depreciados" }

[ ] Tentar depreciar flow "erro"
    Entrada: POST /flows/5/deprecate
    Esperado: HTTP 409 · { error: "Apenas flows publicados podem ser depreciados" }

[ ] Tentar depreciar flow já "depreciado"
    Entrada: POST /flows/4/deprecate
    Esperado: HTTP 409 · { error: "Flow já está depreciado" }

[ ] Tentar depreciar flow inexistente
    Entrada: POST /flows/999/deprecate
    Esperado: HTTP 404 · { error: "Flow não encontrado" }

[ ] Após depreciar — não pode ser republicado nem editado
    Entrada: POST /flows/1/deprecate (sucesso) → POST /flows/1/publish
    Esperado: Primeira chamada retorna 200 · Segunda retorna 409

[ ] Flow "depreciado" disponibiliza as ações: visualizar, duplicar e excluir
    Entrada: GET /flows/4  (status "depreciado")
    Esperado: HTTP 200 · ações disponíveis = [visualizar, duplicar, excluir]

[ ] Após depreciar — flow permanece consultável (GET)
    Entrada: POST /flows/1/deprecate (sucesso) → GET /flows/1
    Esperado: GET retorna 200 · { status: "depreciado" }
```

---

## 7. Testes de Ação: Duplicar — `POST /flows/:id/duplicate`

```
[ ] Duplicar flow "publicado"
    Entrada: POST /flows/1/duplicate  (flow 1: "Coleta de dados Syscob")
    Esperado: HTTP 201 · { id: <novo_id>, name: "Coleta de dados Syscob (cópia)", status: "rascunho" }

[ ] Duplicar flow "rascunho"
    Entrada: POST /flows/2/duplicate
    Esperado: HTTP 201 · { name: "Formulário de negociação (cópia)", status: "rascunho" }

[ ] Duplicar flow "erro"
    Entrada: POST /flows/5/duplicate
    Esperado: HTTP 201 · { name: "Validação de CPF (cópia)", status: "rascunho" }

[ ] Duplicar flow "depreciado"
    Entrada: POST /flows/4/duplicate
    Esperado: HTTP 201 · { name: "<nome original> (cópia)", status: "rascunho" }

[ ] Duplicar flow — cópia contém todos os campos do original
    Entrada: POST /flows/1/duplicate  (flow 1 tem 3 campos)
    Esperado: HTTP 201 · { fields: [<3 campos com mesmos type, label, required, options>] }

[ ] Duplicar flow — cópia recebe novo ID distinto do original
    Entrada: POST /flows/1/duplicate
    Esperado: HTTP 201 · { id: <valor diferente de 1> }

[ ] Duplicar flow — cópia recebe createdAt com data/hora atual
    Entrada: POST /flows/1/duplicate
    Esperado: HTTP 201 · { createdAt: <datetime próximo ao momento da chamada> }

[ ] Duplicar flow — alterações na cópia não afetam o original
    Entrada: POST /flows/1/duplicate (obtém id_copia) → PATCH /flows/<id_copia> { name: "Alterado" }
    Esperado: GET /flows/1 → { name: "Coleta de dados Syscob" } (original inalterado)

[ ] Duplicar flow inexistente
    Entrada: POST /flows/999/duplicate
    Esperado: HTTP 404 · { error: "Flow não encontrado" }
```

---

## 8. Testes de Campos — `CRUD em /flows/:id/fields`

```
[ ] Listar campos de um flow existente
    Entrada: GET /flows/1/fields
    Esperado: HTTP 200 · [ { id, flowId, type, label, required, order, options: [] } ]

[ ] Listar campos de flow inexistente
    Entrada: GET /flows/999/fields
    Esperado: HTTP 404 · { error: "Flow não encontrado" }

[ ] Criar campo do tipo "short" (Resposta curta) em flow "rascunho"
    Entrada: POST /flows/2/fields · { type: "short", label: "Nome completo", inputType: "text", required: true }
    Esperado: HTTP 201 · { id: <uuid>, flowId: 2, type: "short", label: "Nome completo", inputType: "text", required: true, order: <próximo>, options: [] }

[ ] Criar campo "short" com cada subtipo (inputType) válido
    Entrada: POST /flows/2/fields · { type: "short", label: "Campo", inputType: <"text"|"email"|"number"|"phone"|"password"|"passcode"> }
    Esperado: HTTP 201 · { type: "short", inputType: <valor informado> }

[ ] Criar campo "short" com inputType inválido
    Entrada: POST /flows/2/fields · { type: "short", label: "Campo", inputType: "cpf" }
    Esperado: HTTP 422 · { errors: [{ field: "inputType", message: "Tipo de entrada inválido" }] }

[ ] Criar campo do tipo "text" (Resposta longa) em flow "rascunho"
    Entrada: POST /flows/2/fields · { type: "text", label: "Comentários", required: false }
    Esperado: HTTP 201 · { type: "text", options: [] }  (texto livre, sem subtipo)

[ ] Criar campo do tipo "consent" (Campo de consentimento)
    Entrada: POST /flows/2/fields · { type: "consent", label: "Aceito os termos de uso", required: true }
    Esperado: HTTP 201 · { type: "consent", options: [] }

[ ] Adicionar instrução opcional a campo short/text/date
    Entrada: PATCH /flows/2/fields/1 · { instruction: "Informe seu nome completo" }
    Esperado: HTTP 200 · { instruction: "Informe seu nome completo" }

[ ] Criar campo do tipo "radio" com 2 opções
    Entrada: POST /flows/2/fields · { type: "radio", label: "Motivo", required: true, options: ["Negociação", "Dúvida"] }
    Esperado: HTTP 201 · { type: "radio", options: [{ value: "Negociação", order: 1 }, { value: "Dúvida", order: 2 }] }

[ ] Criar campo do tipo "checkbox" com 2 opções
    Entrada: POST /flows/2/fields · { type: "checkbox", label: "Canais", required: false, options: ["WhatsApp", "E-mail"] }
    Esperado: HTTP 201 · { type: "checkbox", options: [{ value: "WhatsApp" }, { value: "E-mail" }] }

[ ] Criar campo do tipo "dropdown" com 3 opções
    Entrada: POST /flows/2/fields · { type: "dropdown", label: "Departamento", required: true, options: ["Financeiro", "Comercial", "Suporte"] }
    Esperado: HTTP 201 · { type: "dropdown", options: [<3 opções>] }

[ ] Criar campo do tipo "date"
    Entrada: POST /flows/2/fields · { type: "date", label: "Data de nascimento", required: false }
    Esperado: HTTP 201 · { type: "date", options: [] }

[ ] Criar campo sem label — erro de validação
    Entrada: POST /flows/2/fields · { type: "text", label: "", required: true }
    Esperado: HTTP 422 · { errors: [{ field: "label", message: "Label do campo é obrigatório" }] }

[ ] Criar campo "radio" com apenas 1 opção
    Entrada: POST /flows/2/fields · { type: "radio", label: "Escolha", options: ["Apenas uma"] }
    Esperado: HTTP 422 · { errors: [{ field: "options", message: "Campos deste tipo exigem ao menos 2 opções" }] }

[ ] Criar campo "dropdown" sem opções
    Entrada: POST /flows/2/fields · { type: "dropdown", label: "Selecione", options: [] }
    Esperado: HTTP 422 · { errors: [{ field: "options", message: "Campos deste tipo exigem ao menos 2 opções" }] }

[ ] Criar campo em flow "publicado" — bloqueado
    Entrada: POST /flows/1/fields · { type: "text", label: "Novo campo" }
    Esperado: HTTP 409 · { error: "Flows publicados não podem ser editados" }

[ ] Criar campo em flow "depreciado" — bloqueado
    Entrada: POST /flows/4/fields · { type: "text", label: "Novo campo" }
    Esperado: HTTP 409 · { error: "Flows depreciados não podem ser editados" }

[ ] Atualizar label de um campo
    Entrada: PATCH /flows/2/fields/1 · { label: "Nome e sobrenome" }
    Esperado: HTTP 200 · { label: "Nome e sobrenome" }

[ ] Atualizar required de um campo
    Entrada: PATCH /flows/2/fields/1 · { required: false }
    Esperado: HTTP 200 · { required: false }

[ ] Atualizar order de campo — reordenação
    Entrada: PATCH /flows/2/fields · { order: [fieldId3, fieldId1, fieldId2] }
    Esperado: HTTP 200 · campos retornam com a nova ordenação

[ ] Remover campo de flow em edição
    Entrada: DELETE /flows/2/fields/1
    Esperado: HTTP 200 ou 204 · GET /flows/2/fields não retorna o campo removido

[ ] Remover campo inexistente
    Entrada: DELETE /flows/2/fields/999
    Esperado: HTTP 404 · { error: "Campo não encontrado" }

[ ] Adicionar opção a campo existente
    Entrada: POST /flows/2/fields/3/options · { value: "Nova opção" }
    Esperado: HTTP 201 · { id: <uuid>, fieldId: 3, value: "Nova opção", order: <próximo> }

[ ] Remover opção de campo com 3 opções — permitido
    Entrada: DELETE /flows/2/fields/3/options/2  (campo com 3 opções)
    Esperado: HTTP 200 ou 204 · campo passa a ter 2 opções

[ ] Remover opção de campo com apenas 2 opções — bloqueado
    Entrada: DELETE /flows/2/fields/3/options/2  (campo com 2 opções)
    Esperado: HTTP 422 · { error: "Mínimo de 2 opções para este tipo de campo" }

[ ] Adicionar 6ª opção em campo "radio" — bloqueado (máximo 5)
    Entrada: POST /flows/2/fields/3/options · { value: "Sexta" }  (campo radio com 5 opções)
    Esperado: HTTP 422 · { error: "Máximo de 5 opções em Seleção única" }

[ ] Máximo de 5 opções não se aplica a "checkbox" nem "dropdown"
    Entrada: POST /flows/2/fields/4/options · { value: "Sexta" }  (campo checkbox com 5 opções)
    Esperado: HTTP 201 · campo passa a ter 6 opções
```

---

## 9. Testes de Validação de Negócio

```
[ ] Validar que flow sem number não pode ser publicado
    Entrada: Criar flow sem number → POST /flows/:id/publish
    Esperado: HTTP 422 · { errors: [{ field: "number" }] }

[ ] Validar que flow com number inválido é rejeitado na criação
    Entrada: POST /flows · { number: "numero_invalido", ...demais campos válidos }
    Esperado: HTTP 422 · { errors: [{ field: "number", message: "Número inválido" }] }

[ ] Validar que "confirmationMessage" é opcional — criação sem o campo
    Entrada: POST /flows · { name, number, title, submitButtonText } (sem confirmationMessage)
    Esperado: HTTP 201 (sem erro de validação)

[ ] Validar que campo "text" não exige options
    Entrada: POST /flows/:id/fields · { type: "text", label: "Campo", required: true }
    Esperado: HTTP 201 · { options: [] }

[ ] Validar que campo "date" não exige options
    Entrada: POST /flows/:id/fields · { type: "date", label: "Data", required: false }
    Esperado: HTTP 201 · { options: [] }

[ ] Validar que tipo de campo inválido é rejeitado
    Entrada: POST /flows/:id/fields · { type: "invalid_type", label: "Campo" }
    Esperado: HTTP 422 · { errors: [{ field: "type", message: "Tipo de campo inválido" }] }
    # enum válido: short · text · radio · checkbox · dropdown · date · consent

[ ] Validar que campos short, consent e text não exigem options
    Entrada: POST /flows/:id/fields · { type: "consent", label: "Aceito os termos" }
    Esperado: HTTP 201 · { options: [] }

[ ] Validar que options com valores em branco são considerados ausentes para radio/checkbox/dropdown
    Entrada: POST /flows/:id/publish  (campo radio com options: ["", ""])
    Esperado: HTTP 422 · { errors: [{ message: "Campo exige ao menos 2 opções preenchidas" }] }

[ ] Validar que order dos campos é sequencial após criação
    Entrada: Criar 3 campos sequencialmente
    Esperado: campos retornados com order: [1, 2, 3]

[ ] Validar integridade referencial — campo pertence ao flow correto
    Entrada: PATCH /flows/2/fields/99  (campo 99 pertence ao flow 3)
    Esperado: HTTP 404 · { error: "Campo não encontrado" }

[ ] Validar que status só aceita valores do enum permitido
    Entrada: PATCH /flows/2 · { status: "ativo" }
    Esperado: HTTP 422 · { errors: [{ field: "status", message: "Status inválido" }] }
```

---

## 10. Testes de Transição de Status

```
[ ] Transição válida: rascunho → publicado
    Entrada: Flow com status "rascunho" e dados válidos → POST /flows/:id/publish
    Esperado: HTTP 200 · { status: "publicado" }

[ ] Transição válida: publicado → depreciado
    Entrada: Flow com status "publicado" → POST /flows/:id/deprecate
    Esperado: HTTP 200 · { status: "depreciado" }

[ ] Transição válida: erro → publicado (após correção)
    Entrada: Flow com status "erro" e dados corrigidos → POST /flows/:id/publish
    Esperado: HTTP 200 · { status: "publicado" }

[ ] Transição inválida: rascunho → depreciado
    Entrada: Flow com status "rascunho" → POST /flows/:id/deprecate
    Esperado: HTTP 409 · { error: "Apenas flows publicados podem ser depreciados" }

[ ] Transição inválida: depreciado → publicado
    Entrada: Flow com status "depreciado" → POST /flows/:id/publish
    Esperado: HTTP 409 · { error: "Flows depreciados não podem ser republicados" }

[ ] Transição inválida: depreciado → rascunho (reativação)
    Entrada: Flow com status "depreciado" → PATCH /flows/:id { status: "rascunho" }
    Esperado: HTTP 409 · { error: "Transição de status não permitida" }

[ ] Transição inválida: publicado → rascunho
    Entrada: Flow com status "publicado" → PATCH /flows/:id { status: "rascunho" }
    Esperado: HTTP 409 · { error: "Transição de status não permitida" }

[ ] Transição inválida: erro → depreciado
    Entrada: Flow com status "erro" → POST /flows/:id/deprecate
    Esperado: HTTP 409 · { error: "Apenas flows publicados podem ser depreciados" }

[ ] Status "depreciado" é estado final — nenhuma transição de saída
    Entrada: Flow com status "depreciado" → qualquer endpoint de ação (/publish, /deprecate, PATCH)
    Esperado: HTTP 409 em todas as tentativas

[ ] Duplicar flow publicado gera cópia em "rascunho" — não afeta o original
    Entrada: POST /flows/1/duplicate
    Esperado: { status: "rascunho" } na cópia · GET /flows/1 → { status: "publicado" } (original)

[ ] Duplicar flow com status "erro" gera cópia em "rascunho"
    Entrada: POST /flows/5/duplicate
    Esperado: { status: "rascunho" } · status "erro" não é propagado para a cópia
```

---

## 11. Testes de Busca, Filtro e Paginação

```
[ ] Busca por nome — correspondência parcial, case-insensitive
    Entrada: GET /flows?search=coleta
    Esperado: HTTP 200 · data contém flows cujo name inclui "coleta" (ignorando maiúsculas)

[ ] Busca por nome — sem correspondência
    Entrada: GET /flows?search=xyzabc123
    Esperado: HTTP 200 · { data: [], total: 0 }

[ ] Filtro por número — retorna apenas flows do número informado
    Entrada: GET /flows?number=11+3777-1887
    Esperado: HTTP 200 · todos os itens de data possuem number "11 3777-1887"

[ ] Filtro por status "publicado"
    Entrada: GET /flows?status=publicado
    Esperado: HTTP 200 · todos os itens de data possuem status "publicado"

[ ] Filtro por status "rascunho"
    Entrada: GET /flows?status=rascunho
    Esperado: HTTP 200 · todos os itens de data possuem status "rascunho"

[ ] Filtro por status "depreciado"
    Entrada: GET /flows?status=depreciado
    Esperado: HTTP 200 · todos os itens de data possuem status "depreciado"

[ ] Filtro por status "erro"
    Entrada: GET /flows?status=erro
    Esperado: HTTP 200 · todos os itens de data possuem status "erro"

[ ] Filtro por status inválido
    Entrada: GET /flows?status=invalido
    Esperado: HTTP 422 · { error: "Status inválido" } OU HTTP 200 · { data: [], total: 0 }

[ ] Combinação: busca + filtro por número
    Entrada: GET /flows?search=coleta&number=11+3777-1887
    Esperado: HTTP 200 · data contém apenas flows com "coleta" no nome E número "11 3777-1887"

[ ] Combinação: filtro por número + filtro por status
    Entrada: GET /flows?number=11+3777-9335&status=publicado
    Esperado: HTTP 200 · data contém apenas flows do número "11 3777-9335" com status "publicado"

[ ] Ordenação por mais recentes (padrão ou explícito)
    Entrada: GET /flows?sort=recentes  (com múltiplos flows de datas variadas)
    Esperado: HTTP 200 · data[0].createdAt >= data[1].createdAt >= ...

[ ] Ordenação por mais antigos
    Entrada: GET /flows?sort=antigos
    Esperado: HTTP 200 · data[0].createdAt <= data[1].createdAt <= ...

[ ] Ordenação por nome A–Z
    Entrada: GET /flows?sort=nome
    Esperado: HTTP 200 · data retornado em ordem alfabética pelo campo name (locale pt-BR)

[ ] Paginação — primeira página com 10 itens (25 registros no banco)
    Entrada: GET /flows?page=1&perPage=10
    Esperado: HTTP 200 · { data: [<10 itens>], total: 25, page: 1, perPage: 10, totalPages: 3 }

[ ] Paginação — segunda página com 10 itens
    Entrada: GET /flows?page=2&perPage=10
    Esperado: HTTP 200 · { data: [<10 itens>], page: 2 } · itens distintos da página 1

[ ] Paginação — terceira página com 5 itens restantes (25 registros)
    Entrada: GET /flows?page=3&perPage=10
    Esperado: HTTP 200 · { data: [<5 itens>], total: 25, page: 3 }

[ ] Paginação com 25 itens por página
    Entrada: GET /flows?page=1&perPage=25  (30 registros no banco)
    Esperado: HTTP 200 · { data: [<25 itens>], total: 30, perPage: 25, totalPages: 2 }

[ ] Paginação com 50 itens por página
    Entrada: GET /flows?page=1&perPage=50  (60 registros no banco)
    Esperado: HTTP 200 · { data: [<50 itens>], total: 60, perPage: 50, totalPages: 2 }

[ ] Página inexistente retorna lista vazia (não erro)
    Entrada: GET /flows?page=999&perPage=10  (apenas 25 registros no banco)
    Esperado: HTTP 200 · { data: [], total: 25, page: 999 }

[ ] Paginação combinada com filtro
    Entrada: GET /flows?status=publicado&page=1&perPage=10  (15 flows publicados)
    Esperado: HTTP 200 · { data: [<10 itens>], total: 15, totalPages: 2 }

[ ] Paginação — perPage inválido é rejeitado ou usa valor padrão
    Entrada: GET /flows?perPage=999
    Esperado: HTTP 422 · { error: "perPage deve ser 10, 25 ou 50" }
    OU: HTTP 200 com perPage usando valor padrão (10)

[ ] Paginação reseta para página 1 quando total de resultados muda com novo filtro
    Entrada: GET /flows?page=3&perPage=10 → GET /flows?status=publicado&page=1&perPage=10
    Esperado: a segunda chamada retorna resultados filtrados a partir da página 1
```

---

## 12. Testes de Limites de Caracteres

```
# Limites por tipo:
#   label short/text/date         → 20
#   label radio/checkbox/dropdown → 30
#   label consent (descrição)     → 300
#   título de opção               → 30
#   submitButtonText              → 30
#   instruction (opcional)        → 80

[ ] Label de campo short acima de 20 caracteres
    Entrada: PATCH /flows/2/fields/1 · { label: "Nome completo do cliente aqui" }  (type short)
    Esperado: HTTP 422 · { errors: [{ field: "label", message: "Máximo de 20 caracteres" }] }

[ ] Label de campo radio acima de 30 caracteres
    Entrada: POST /flows/2/fields · { type: "radio", label: "<31+ caracteres>", options: ["A", "B"] }
    Esperado: HTTP 422 · { errors: [{ field: "label", message: "Máximo de 30 caracteres" }] }

[ ] Título de opção acima de 30 caracteres
    Entrada: POST /flows/2/fields · { type: "dropdown", label: "Estado", options: ["<31+ caracteres>", "SP"] }
    Esperado: HTTP 422 · { errors: [{ field: "options", message: "Máximo de 30 caracteres por opção" }] }

[ ] Descrição (label) de campo consent acima de 300 caracteres
    Entrada: POST /flows/2/fields · { type: "consent", label: "<301+ caracteres>" }
    Esperado: HTTP 422 · { errors: [{ field: "label", message: "Máximo de 300 caracteres" }] }

[ ] submitButtonText acima de 30 caracteres
    Entrada: PATCH /flows/2 · { submitButtonText: "<31+ caracteres>" }
    Esperado: HTTP 422 · { errors: [{ field: "submitButtonText", message: "Máximo de 30 caracteres" }] }

[ ] instruction acima de 80 caracteres
    Entrada: PATCH /flows/2/fields/1 · { instruction: "<81+ caracteres>" }
    Esperado: HTTP 422 · { errors: [{ field: "instruction", message: "Máximo de 80 caracteres" }] }

[ ] Valores dentro do limite são aceitos
    Entrada: campos com label/opção/instrução/botão exatamente no limite máximo
    Esperado: HTTP 200/201 (sem erro de validação)
```

---

## 13. Testes de Erro de Publicação (retorno da Meta)

```
[ ] Publicação rejeitada pela Meta — flow vai para status "erro"
    Entrada: POST /flows/2/publish  (validação interna OK, Meta retorna erro)
    Esperado: HTTP 200/422 · { status: "erro", publishError: { reasons: [<lista de motivos>] } }
    # Front exibe status "Erro de publicação"

[ ] Resposta de erro inclui título, subtítulo e lista de motivos
    Entrada: POST /flows/2/publish (rejeitado pela Meta)
    Esperado: payload com:
      title: "Erro de publicação"
      subtitle: "Não foi possível realizar a publicação. Verifique e corrija os motivos retornados e tente novamente."
      reasons: [ "JSON do Flow inválido...", "Categoria incompatível...", "WABA não verificado...", ... ]

[ ] Flow em "erro" pode ser corrigido e republicado com sucesso
    Entrada: PATCH /flows/2 (corrige dados) → POST /flows/2/publish (Meta aceita)
    Esperado: HTTP 200 · { status: "publicado" }

[ ] Flow em "erro" mantém ações: editar, excluir, duplicar
    Entrada: GET /flows/2  (status "erro")
    Esperado: HTTP 200 · ações disponíveis = [editar, excluir, duplicar]
```

---

## 14. Testes de Ações por Status (listagem)

```
# Matriz de ações por status:
#   rascunho    → editar, excluir, duplicar
#   erro        → editar, excluir, duplicar
#   publicado   → visualizar, depreciar, duplicar
#   depreciado  → visualizar, duplicar, excluir

[ ] Ações de flow "rascunho"
    Entrada: GET /flows/2  (status "rascunho")
    Esperado: ações = [editar, excluir, duplicar]

[ ] Ações de flow "erro"
    Entrada: GET /flows/5  (status "erro")
    Esperado: ações = [editar, excluir, duplicar]

[ ] Ações de flow "publicado"
    Entrada: GET /flows/1  (status "publicado")
    Esperado: ações = [visualizar, depreciar, duplicar]

[ ] Ações de flow "depreciado"
    Entrada: GET /flows/4  (status "depreciado")
    Esperado: ações = [visualizar, duplicar, excluir]

[ ] Flow "publicado" não expõe ação de editar nem excluir
    Entrada: GET /flows/1
    Esperado: ações não contêm "editar" nem "excluir"

[ ] Flow "rascunho"/"erro" não expõe ação de visualizar nem depreciar
    Entrada: GET /flows/2
    Esperado: ações não contêm "visualizar" (somente leitura) nem "depreciar"
```

---

## 15. Testes do Seletor de Número WhatsApp (WABA)

```
[ ] Seletor lista números com dados completos
    Entrada: abrir seletor "Número do WhatsApp" no construtor
    Esperado: cada item exibe { nome, número, categoria (quando houver), quantidade de contatos }

[ ] Item sem categoria não quebra o layout
    Entrada: número WABA com category vazia
    Esperado: item renderizado sem a linha de categoria

[ ] Selecionar item vincula valor e aciona autosave
    Entrada: clicar no item "Vonex · 11 97699-2942"
    Esperado: campo number = "11 97699-2942" · gatilho de autosave acionado · seletor fechado

[ ] Seletor fecha ao clicar fora
    Entrada: abrir seletor → clicar em área externa
    Esperado: lista é fechada sem alterar a seleção atual
```

---

## 16. Testes do Modo Somente Leitura (Visualizar)

```
[ ] Visualizar flow "publicado" abre construtor somente leitura
    Entrada: acionar "Visualizar" em flow publicado
    Esperado: todos os inputs/selects desabilitados · botões "Salvar rascunho" e "Publicar flow" ocultos

[ ] Visualizar flow "depreciado" abre construtor somente leitura
    Entrada: acionar "Visualizar" em flow depreciado
    Esperado: conteúdo consultável · nenhuma edição permitida

[ ] Modo somente leitura oculta ações de campo
    Entrada: construtor em modo visualização
    Esperado: ocultos adicionar campo, excluir campo, alça de reordenar, adicionar/excluir opção · toggle e seletor de número não interativos

[ ] Editar (não visualizar) mantém construtor editável
    Entrada: acionar "Editar" em flow "rascunho" ou "erro"
    Esperado: inputs habilitados · ações de edição visíveis · sem classe de somente leitura
```

---

## 17. Testes do Checklist de Publicação ("Etapas para publicar")

```
[ ] Botão "Publicar flow" desabilitado com pendências
    Entrada: flow com ao menos 1 item obrigatório vazio
    Esperado: botão "Publicar flow" disabled

[ ] Botão "Publicar flow" habilitado sem pendências
    Entrada: flow com todos os obrigatórios preenchidos e campos válidos
    Esperado: botão "Publicar flow" habilitado

[ ] Checklist agrupa pendências em "Configurações gerais" e "Campos da tela"
    Entrada: hover no botão "Publicar flow" com pendências
    Esperado: popover exibe as duas categorias com indicador concluída/pendente e subcategorias

[ ] Subcategorias de "Campos da tela" são dinâmicas por tipo de campo
    Entrada: campo "short" sem título + campo "radio" com <2 opções
    Esperado: lista "Preencha ou exclua o campo de resposta curta" e "Adicione ao menos 2 opções no campo de seleção única"

[ ] Mensagens pluralizam quando há múltiplos campos do mesmo tipo
    Entrada: 2 campos "short" sem título
    Esperado: "Preencha ou exclua os campos de resposta curta" (plural)

[ ] Contagem de pendências reflete o total das categorias
    Entrada: flow em branco
    Esperado: contagem = soma dos itens de "Configurações gerais" + "Campos da tela"
```

## 18. Componentes de Exibição e Aceite (CCM-3070)

```
[ ] Picker agrupa componentes em "Exibição" e "Entradas"
    Entrada: abrir "Adicionar componente"
    Esperado: grupo Exibição (Imagem, Carrossel, Título, Subtítulo, Legenda, Texto formatado) e Entradas (…, Aceite)

[ ] Imagem por upload converte para base64 e mostra na prévia
    Entrada: addField('image') + uploadImg
    Esperado: f.img preenchido (data URI); <.wa-image img> presente na prévia

[ ] Imagem por URL válida (.jpg/.png) carrega
    Entrada: origem URL + validateImgUrl com https://x/y.jpg
    Esperado: f.img preenchido; toast de sucesso

[ ] Imagem por URL inválida bloqueia com mensagem
    Entrada: validateImgUrl com "abc"
    Esperado: f.img vazio; toast "Não foi possível carregar esta imagem."

[ ] Carrossel: 2 a 3 imagens; addCarouselImg limita a 3, removeCarouselImg mantém min 2
    Esperado: 4ª imagem bloqueada; botão "Adicionar imagem" desabilitado em 3 (sem texto "máx. 3"); remover abaixo de 2 bloqueado

[ ] Máximo de 3 carrosséis por Flow
    Entrada: addField('carousel') 4x
    Esperado: 4º bloqueado com toast "Máximo de 3 carrosséis por Flow."; validate()/checklist também acusam

[ ] Barra de formatação aplica negrito/itálico à seleção
    Entrada: selecionar trecho + rtFormat(id,'b') e rtFormat(id,'i')
    Esperado: conteúdo vira **trecho** e ***trecho***; sem texto de apoio abaixo do campo

[ ] Carrossel <2 imagens válidas bloqueia publicação
    Esperado: checklist "Adicione ao menos duas imagens ao carrossel"

[ ] Texto alternativo obrigatório por imagem no carrossel
    Esperado: checklist "Preencha o texto alternativo de cada imagem do carrossel"

[ ] Texto formatado renderiza negrito/itálico/lista/link na prévia
    Entrada: content "**b**\n- item\n[texto](url)"
    Esperado: <b>, <li>, .rt-link na .wa-richtext

[ ] Regra 8.2 — pelo menos um campo de entrada
    Entrada: tela só com componentes de exibição
    Esperado: checklist e validate() "Adicione ao menos um campo de entrada à tela…"; Aceite conta como entrada

[ ] Componentes de exibição não têm toggle "obrigatório"
    Esperado: image/carousel/heading/subheading/caption/richtext sem toggle; badge "Obrigatório" só em entradas

[ ] Aceite: texto até 120 chars + "Obrigatório para enviar"; prévia com caixa de marcar
    Esperado: LABEL_MAX.consent = 120; .wa-check-box na prévia

[ ] Limites dos textos de exibição
    Entrada: digitar acima do limite em cada tipo
    Esperado: Título e Subtítulo até 80; Legenda e Texto formatado até 4096; maxlength + contador (xc_) atualizado e truncamento em setContent

[ ] Reposicionar/excluir componentes de exibição (mesmo padrão dos campos)
    Esperado: moveField e removeField funcionam para os novos tipos
```

---

*Documento gerado em 28/05/2026 · Atualizado em 10/07/2026 — Referências: CCM-2743, CCM-3070*
