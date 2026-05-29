# De-Para: Protótipo → Angular (ngx-ccm)

> Este documento mapeia cada elemento do protótipo HTML/CSS para seu equivalente
> exato no projeto Angular `ngx-ccm-main`. É a referência primária para o
> desenvolvedor front-end Angular que vai reimplementar esta feature.
>
> Projeto Angular base: `src/app/` em `ngx-ccm-main`
> Protótipo: `index.html` + `app.css`

---

## 1. TOKENS CSS — Variáveis de Cor

| Variável no Protótipo | Variável no Angular (ngx-ccm) | Valor Hex | Significado |
|---|---|---|---|
| `--clr-violet` | `--brand-color-primary-pure` | `#9240FF` | Cor primária da marca (violet/purple) |
| `--clr-violet-h` | `--brand-color-primary-medium` | `#8766FF` | Primária hover |
| `--clr-violet-d` | `--brand-color-primary-dark` | `#410293` | Primária escura |
| `--clr-violet-l` | `--brand-color-primary-light` | `#D7CCFF` | Primária clara |
| `--clr-violet-ll` | `--brand-color-primary-lightest` | `#EBE5FF` | Primária mais clara (fundos) |
| `--clr-violet-dm` | `--brand-color-primary-dark-medium` | `#5F3AE5` | Primária escura média |
| `--clr-cyan` | `--brand-color-secundary-pure` | `#37B8FB` | Cor secundária (cyan/sky blue) |
| `--clr-cyan-l` | `--brand-color-secundary-light` | `#BFE6FA` | Secundária clara |
| `--clr-cyan-ll` | `--brand-color-secundary-lightest` | `#E8F7FF` | Secundária mais clara |
| `--clr-cyan-md` | `--brand-color-secundary-medium` | `#3A8FB9` | Secundária média |
| `--clr-cyan-d` | `--brand-color-secundary-dark` | `#114865` | Secundária escura |
| `--grad` | `linear-gradient(90deg, var(--brand-color-gradient-start) 0%, var(--brand-color-gradient-end) 100%)` | `#1FF0FF → #9240FF` | Gradiente de marca |
| `--bg` | `--background-color-light` | `#f2f6fa` | Background geral da página |
| `--surface` | `--background-color-lightest` | `#ffffff` | Background de cards/superfícies |
| `--t1` | `--neutral-color-low-pure` | `#28293d` | Texto primário (mais escuro) |
| `--t2` | `--neutral-color-low-medium` | `#555770` | Texto secundário |
| `--t3` | `--neutral-color-low-light` | `#9091af` | Texto terciário / placeholders |
| `--tph` | `--neutral-color-high-dark` | `#c7c9d9` | Placeholder de inputs |
| `--border` | `--neutral-color-high-medium` | `#d6e1e9` | Borda padrão |
| `--border-s` | `--neutral-color-high-light` | `#f1f1f5` | Borda sutil / separadores internos |
| `--border-d` | `--neutral-color-high-dark` | `#c7c9d9` | Borda forte |
| `--ok` | `--feedback-color-success-pure` | `#79c22f` | Verde sucesso |
| `--ok-bg` | `--feedback-color-success-light` | `#e6f3e5` | Fundo verde sucesso |
| `--ok-t` | `--feedback-color-success-dark` | `#104700` | Texto em fundo verde |
| `--err` | `--feedback-color-warning-pure` | `#f54336` | Vermelho erro |
| `--err-bg` | `--feedback-color-warning-light` | `#ffdde3` | Fundo vermelho erro |
| `--err-t` | `--feedback-color-warning-dark` | `#650012` | Texto em fundo vermelho |
| `--warn` | `--feedback-color-helper-pure` | `#f99f18` | Laranja aviso |
| `--warn-bg` | `--feedback-color-helper-light` | `#fff3e0` | Fundo laranja aviso |
| `--warn-t` | `--feedback-color-helper-dark` | `#734700` | Texto em fundo laranja |

> **Arquivo Angular de referência:** `src/app/@theme/style-guide/css-files/colors.scss`

---

## 2. TOKENS CSS — Sombras

| Uso no Protótipo | Classe/Token Angular | Valor CSS |
|---|---|---|
| `--shadow` (cards) | `.ccm-card` | `box-shadow: 0px 0px 10px 0px rgba(40, 41, 61, 0.10)` |
| `--shadow-lg` (modais, popovers) | `.ccm-popover` | `box-shadow: 0px 4px 20px 0px rgba(0,0,0,.12), 0px 2px 4px 0px rgba(0,0,0,.12)` |
| Sombra direcional `↑` | `.shadow-up` | `box-shadow: 0px -8px 8px 0px rgba(0,0,0,.05)` |
| Sombra direcional `↓` | `.shadow-down` | `box-shadow: 0px 8px 8px 0px rgba(0,0,0,.05)` |

---

## 3. TOKENS CSS — Border Radius

| Variável no Protótipo | Classe Angular Equivalente | Valor |
|---|---|---|
| `--r-sm: 8px` | `.br-8` | `border-radius: 8px` |
| `--r-md: 12px` | `.br-12` | `border-radius: 12px` |
| `--r-lg: 16px` | `.br-16` | `border-radius: 16px` |
| `--r-xl: 20px` | `.br-20` ou `.white-box-background` | `border-radius: 20px` |
| `--r-pill: 26px` | `.br-26` | `border-radius: 26px` |
| `--r-circ: 9999px` | `border-radius: 50%` ou `.br-100` | `border-radius: 9999px` |

> **Nota:** Classes `.br-0` a `.br-100` são geradas dinamicamente em `src/styles.scss`
> via loop SCSS `@for $i from 0 through 100`.

---

## 4. TIPOGRAFIA

| Uso no Protótipo | Classe Angular | Tamanho / Peso |
|---|---|---|
| Título da página (h3) | `<h3>` nativo | `20px / weight 500` |
| Subtítulo da página | `<h5>` ou `.body-xx-small-regular` | `14px / weight 400` |
| Label de campo (`NOME DO FLOW *`) | `.caption-xxxx-small-semi-bold` | `12px / weight 600` |
| Texto de botão | `.body-xx-small-semi-bold` | `14px / weight 600` |
| Texto de célula de tabela | `.body-xx-small-regular` ou `<h5>` | `14px / weight 400` |
| Meta texto ("3 campos") | `.caption-xxx-small` | `10px / weight 400` |
| Badge/chip texto | `.caption-xxx-small-semi-bold` | `10px / weight 600` |
| Texto do header de coluna | `.caption-xxxx-small-semi-bold` | `12px / weight 600` |
| Texto de cabeçalho de seção | `<h4>` ou `.body-x-small-semi-bold` | `16px / weight 600` |

> **Fonte:** Montserrat (local em `assets/fonts/Montserrat-Regular.ttf`)
> **Arquivo Angular de referência:** `src/app/@theme/style-guide/css-files/typography.scss`

---

## 5. COMPONENTES — De-Para Principal

### 5.1 Layout Shell

| Elemento no Protótipo | Componente Angular | Seletor | Localização |
|---|---|---|---|
| `<header class="hdr">` | Header principal | `<app-header>` | `src/app/@core/components/header/` |
| `<aside class="sidebar">` | Parte do layout shell (roteado) | — | Gerado pelo módulo de routing |
| Logo `vonex_ai.svg` | Parte do `<app-header>` | — | Asset gerenciado no header |
| Nav menu (`Campanha`, `Chat`…) | `<app-header>` interno | — | `header-menu.service.ts` |

> **Nota:** O `<app-header>` da codebase Angular já encapsula toda a barra superior,
> incluindo logo, navegação, notificações, avatar e ações rápidas.
> A sidebar é gerada pelo layout do módulo pai.

---

### 5.2 Botões → `<ccm-button>`

**Componente:** `src/app/@shared/components/button/button.component.ts`
**Seletor:** `<ccm-button>`
**Standalone:** sim

| Classe no Protótipo | Tipo Angular | Código Angular |
|---|---|---|
| `.btn-prim` | `primary` | `<ccm-button type="primary">` |
| `.btn-sec` | `secondary` | `<ccm-button type="secondary">` |
| `.btn-quat` | `quaternary` | `<ccm-button type="quaternary">` |
| `.btn-ghost` / `.btn-link` | `terciary` | `<ccm-button type="terciary">` |
| `.btn-danger` (vermelho) | `primary` + forced bg | `<ccm-button type="primary" [forcedBackground]="'var(--feedback-color-warning-pure)'">`|
| `.btn-warn` (laranja) | `primary` + forced bg | `<ccm-button type="primary" [forcedBackground]="'var(--feedback-color-helper-pure)'">`|
| `.btn-lg` (height 40px) | `medium` | `<ccm-button medium>` |
| `.btn-sm` (height 30px) | `small` | `<ccm-button small>` |
| Botão iconico pequeno (`.ico-btn`) | `mini` | `<ccm-button mini type="quinary">` |

**Tipos disponíveis:** `primary` | `secondary` | `terciary` | `quaternary` | `quinary`

**Exemplo completo:**
```html
<!-- Protótipo -->
<button class="btn btn-prim btn-lg">
  <i class="ph ph-plus"></i> Novo flow
</button>

<!-- Angular -->
<ccm-button type="primary" medium (ccmClick)="createFlow()">
  <i class="ph ph-plus"></i>
  <p class="body-xx-small-semi-bold">Novo flow</p>
</ccm-button>
```

---

### 5.3 Modais de Confirmação → `app-simple-confirmation-modal`

**Componente:** `src/app/@shared/modals/simple-confirmation-modal/`
**Abertura via:** `MatDialog.open(SimpleConfirmationModalComponent, { data: ISimpleConfirmationModalData })`

| Campo do protótipo | Interface Angular | Tipo |
|---|---|---|
| Título do modal | `data.title` | `string` |
| Texto do corpo | `data.content` | `string` (suporta HTML) |
| Largura | `data.modalWidth` | `string` (ex: `'480px'`) |
| Mostrar botão fechar | `data.showHeaderCloseBtn` | `boolean` |
| Label botão cancelar | `data.buttonConfig.cancelLabel` | `string` |
| Label botão confirmar | `data.buttonConfig.confirmLabel` | `string` |
| Tipo botão cancelar | `data.buttonConfig.cancelBtnType` | `IButtonTypes` |
| Ocultar botão cancelar | `data.buttonConfig.hideCancelBtn` | `boolean` |
| Ícone botão confirmar | `data.buttonConfig.confirmIconClass` | `string` (classe ph) |

**Exemplo — Modal de Exclusão:**
```typescript
// Protótipo: openDelete(id)
// Angular:
this.dialog.open(SimpleConfirmationModalComponent, {
  data: {
    title: 'Excluir flow',
    content: `Tem certeza que deseja excluir permanentemente o flow "${flow.name}"?
              Esta ação não pode ser desfeita.`,
    modalWidth: '460px',
    showHeaderCloseBtn: true,
    buttonConfig: {
      cancelLabel: 'Cancelar',
      cancelBtnType: 'terciary',
      confirmLabel: 'Excluir',
    }
  } as ISimpleConfirmationModalData
}).afterClosed().subscribe(result => {
  if (result?.success) this.deleteFlow(id);
});
```

**Exemplo — Modal de Depreciação (com alerta):**
```typescript
this.dialog.open(SimpleConfirmationModalComponent, {
  data: {
    title: 'Depreciar flow',
    content: `Você está prestes a depreciar o flow "${flow.name}".
              <br><br><strong>Atenção:</strong> Este flow está sendo usado em 2
              template(s) e 1 jornada(s). Depreciá-lo impedirá o envio nesses contextos.`,
    modalWidth: '460px',
    showHeaderCloseBtn: true,
    buttonConfig: {
      cancelLabel: 'Cancelar',
      cancelBtnType: 'terciary',
      confirmLabel: 'Depreciar',
    }
  } as ISimpleConfirmationModalData
}).afterClosed().subscribe(result => {
  if (result?.success) this.deprecateFlow(id);
});
```

---

### 5.4 Toasts / Snackbars → `FeedbackService`

**Serviço:** `src/app/@shared/services/feedback.service.ts`
**Componente visual:** `src/app/@shared/components/feedback-snackbar/`
**Posição padrão:** `horizontalPosition: 'end'`, `verticalPosition: 'top'` ← já configurado no Angular

| Uso no Protótipo | Método Angular | Quando usar |
|---|---|---|
| `toast('ok', msg)` | `feedbackService.success(msg)` | Ação concluída com êxito |
| `toast('err', msg)` | `feedbackService.error(msg)` | Falha ou erro da API |
| `toast('warn', msg)` | `feedbackService.warn(msg)` | Aviso não-bloqueante |
| `toast('info', msg)` | `feedbackService.neutral(msg)` | Informação neutra |

**Exemplo:**
```typescript
// Protótipo: toast('ok', `Flow "${nm}" publicado com sucesso!`)
// Angular:
this.feedbackService.success(`Flow "${flow.name}" publicado com sucesso!`);

// Protótipo: toast('err', errs[0])
// Angular:
this.feedbackService.error(errs[0]);
```

> **Tipos de ícone no snackbar Angular:**
> - `success` → `ph-check-circle` (verde)
> - `error` → `ph-minus-circle` (vermelho)
> - `warning` → `ph-warning-circle` (laranja)
> - `neutral` → `ph-info` (cinza/azul)

---

### 5.5 Autosave Indicator → `<ccm-autosave-indicator>`

**Componente:** `src/app/@shared/components/autosave-indicator/`
**Seletor:** `<ccm-autosave-indicator>`

| Input no protótipo | Input Angular | Tipo | Descrição |
|---|---|---|---|
| Estado "Salvando…" (spinner) | `[loading]="true"` | `boolean` | Exibe spinner girando |
| Estado "Salvo" (check) | `[loading]="false"` | `boolean` | Exibe `ph-floppy-disk-back` |
| Bloqueado (sem campos) | `[disabledByMinRequirements]="true"` | `boolean` | Exibe `ph-warning` + tooltip |
| Texto do tooltip de aviso | `[warningTooltipText]="'...'` | `string` | Default: 'Salvamento automático não ativo.' |

**Exemplo:**
```html
<!-- Protótipo: <div id="autosave" class="autosave">...</div> -->
<!-- Angular: -->
<ccm-autosave-indicator
  [loading]="isSaving"
  [disabledByMinRequirements]="!hasRequiredFields">
</ccm-autosave-indicator>
```

---

### 5.6 Tabela → Angular Material `mat-table`

| Elemento no protótipo | Equivalente Angular | Observação |
|---|---|---|
| `.card > table.dtable` | `<mat-table>` dentro de `.white-box-background` | Usar `dataSource` com `MatTableDataSource` |
| `<thead><tr><th>` | `<ng-container matColumnDef="...">` + `<mat-header-cell>` | |
| `<tbody><tr><td>` | `<mat-cell>` | |
| `.cell-name` (nome + meta) | Célula customizada com `<h5>` + `<p>` | |
| `.badge` (status) | Custom badge com classes do design system | Ver Seção 6 |
| `.acts` (ações) | Coluna de ações com `<ccm-button mini>` | |
| `.tfoot` (paginação) | `<ccm-table-paginator>` | Ver Seção 5.7 |

**Estilo do card da tabela:**
```html
<!-- Protótipo: <div class="card"><table class="dtable">...</table></div> -->
<!-- Angular: -->
<div class="white-box-background">
  <table mat-table [dataSource]="dataSource" class="w-100">
    <!-- colunas aqui -->
  </table>
</div>
```

---

### 5.7 Paginação → `<ccm-table-paginator>`

**Componente:** `src/app/@shared/components/ccm-table-paginator/`
**Seletor:** `<ccm-table-paginator>`

| Input/Output | Tipo | Descrição |
|---|---|---|
| `[currentPage]` | `number` (required) | Página atual (base 0) |
| `[totalPages]` | `number` (required) | Total de páginas |
| `[currentPageItemsQuantity]` | `number` (required) | Itens na página atual |
| `[pageSize]` | `number` (required) | Tamanho da página atual |
| `[totalItems]` | `number` (required) | Total de itens |
| `[pageSizeOptions]` | `number[]` | Opções de tamanho (default: `[5,10,15,20]`) |
| `[hasPageSizeOptions]` | `boolean` | Mostrar seletor de qtd/página |
| `(paginatorChange)` | `EventEmitter<CCMPaginatorOutput>` | Emite ao mudar página/tamanho |

**Exemplo:**
```html
<!-- Protótipo: <div class="tfoot">...</div> -->
<!-- Angular: -->
<ccm-table-paginator
  [currentPage]="currentPage"
  [totalPages]="totalPages"
  [currentPageItemsQuantity]="flows.length"
  [pageSize]="pageSize"
  [totalItems]="totalFlows"
  [pageSizeOptions]="[10, 25, 50]"
  (paginatorChange)="onPageChange($event)">
</ccm-table-paginator>
```

---

### 5.8 Filtros e Selects → Angular Material `mat-select`

| Elemento no Protótipo | Angular | Observação |
|---|---|---|
| `.sel-filter` (dropdown filtro) | `<mat-form-field>` + `<mat-select>` | Customizado em `material-select.scss` |
| `.inp-search` (busca) | `<mat-form-field>` + `<input matInput>` | Com ícone prefix |
| `.sel-wrap` (select no builder) | `<mat-form-field>` + `<mat-select>` | No formulário reativo |
| `.ipt` (input texto) | `<mat-form-field>` + `<input matInput>` | Com `formControlName` |

**Exemplo de filtro:**
```html
<!-- Protótipo: <div class="sel-filter"><select id="filterNum">...</select></div> -->
<!-- Angular: -->
<mat-form-field appearance="outline">
  <mat-select [(ngModel)]="filterNumber" (selectionChange)="onFilter()">
    <mat-option value="">Número</mat-option>
    <mat-option *ngFor="let n of numbers" [value]="n">{{n}}</mat-option>
  </mat-select>
</mat-form-field>
```

---

### 5.9 Toggle → `<mat-slide-toggle>`

| Elemento no Protótipo | Angular | Observação |
|---|---|---|
| `<button class="toggle">` | `<mat-slide-toggle>` | Customizado em `material-toggle.scss` |
| Estado `.on` | `[checked]="field.required"` | Binding bidirecional |
| Evento `onclick="toggleReq()"` | `(change)="onToggleRequired($event)"` | |

**Exemplo:**
```html
<!-- Protótipo: <button class="toggle on" id="fr_1" onclick="toggleReq(1)"></button> -->
<!-- Angular: -->
<mat-slide-toggle
  [formControlName]="'required'"
  color="primary">
  Campo obrigatório
</mat-slide-toggle>
```

---

### 5.10 Formulário do Builder → `ReactiveFormsModule`

| Campo no Protótipo | FormControl Angular | Validadores |
|---|---|---|
| `#bName` (nome do flow) | `name: FormControl` | `Validators.required` |
| `#bNumber` (número WA) | `number: FormControl` | `Validators.required` |
| `#bTitle` (título da tela) | `title: FormControl` | `Validators.required` |
| `#bSubmit` (texto botão) | `submitButtonText: FormControl` | `Validators.required` |
| `#bConfirm` (confirmação) | `confirmationMessage: FormControl` | — (opcional) |
| Campos da tela | `fields: FormArray` | Mínimo 1 item |
| Label do campo | `label: FormControl` | `Validators.required` |
| Opções (radio/select) | `options: FormArray` | Mínimo 2 itens |

**Estrutura do FormGroup:**
```typescript
this.flowForm = this.fb.group({
  name: ['', [Validators.required]],
  number: ['', [Validators.required]],
  title: ['', [Validators.required]],
  submitButtonText: ['', [Validators.required]],
  confirmationMessage: [''],
  fields: this.fb.array([], [Validators.minLength(1)])
});
```

---

## 6. BADGES DE STATUS → Classes do Design System

| Status no Protótipo | Classe(s) Angular | CSS vars usadas |
|---|---|---|
| `publicado` (verde) | `.green-box-background` ou custom | `--feedback-color-success-*` |
| `rascunho` (cinza) | `.white-box-background` ou custom | `--neutral-color-high-*` |
| `depreciado` (laranja) | `.yellow-box-background` ou custom | `--feedback-color-helper-*` |
| `erro` (vermelho) | `.red-box-background` ou custom | `--feedback-color-warning-*` |

**Classes de box coloridas disponíveis no Angular:**
```scss
.white-box-background  // bg branca, borda neutra, sombra
.blue-box-background   // bg azul claro, borda azul
.red-box-background    // bg vermelho claro, borda vermelha
.yellow-box-background // bg laranja claro, borda laranja
.green-box-background  // bg verde claro, borda verde
// Todas com border-radius: 12px
```

---

## 7. ÍCONES

O projeto Angular e o protótipo usam a **mesma biblioteca de ícones**:
**Phosphor Icons** (`@phosphor-icons/web@2.1.1`)

Não há mudança necessária nos nomes dos ícones. Exemplos:
```html
<i class="ph ph-plus"></i>
<i class="ph ph-pencil-simple"></i>
<i class="ph ph-trash"></i>
<i class="ph ph-rocket-launch"></i>
<i class="ph ph-floppy-disk"></i>
<i class="ph ph-check-circle"></i>
<i class="ph ph-warning-circle"></i>
<i class="ph ph-caret-down"></i>
<i class="ph ph-arrow-left"></i>
```

> O Angular já importa `regular`, `light` e `bold` em `src/styles.scss`.

---

## 8. CLASSES UTILITÁRIAS — Bootstrap + Custom Loop

O projeto Angular usa **Bootstrap 5** + classes utilitárias próprias geradas via SCSS loop.

| Uso no Protótipo | Classe Angular Equivalente |
|---|---|
| `display:flex` | `.d-flex` (Bootstrap) |
| `align-items:center` | `.align-items-center` (Bootstrap) |
| `justify-content:space-between` | `.justify-content-between` (Bootstrap) |
| `flex:1` | `.flex-auto` ou `style="flex: 1"` |
| `gap: 8px` | `.gap-8` (custom loop) |
| `padding: 16px 24px` | `.py-16.px-24` (custom loop) |
| `margin-top: 4px` | `.mt-4` (custom loop) |
| `font-size: 13px` | `.fs-13` (custom loop) |
| `border-radius: 12px` | `.br-12` (custom loop) |
| `white-space: nowrap` | `.one-line-ellipsis` ou inline |
| `overflow: hidden` | `style="overflow: hidden"` |
| `cursor: pointer` | `.pointer` (global class) |
| `user-select: none` | `.protected-text` (global class) |
| `position: relative` | `.position-relative` (Bootstrap) |
| `position: absolute` | `.position-absolute` (Bootstrap) |

---

## 9. MÓDULO ANGULAR SUGERIDO

O módulo a ser criado para WhatsApp Flows segue a estrutura padrão dos feature modules:

```
src/app/@modules/whatsapp-flows/
├── whatsapp-flows.module.ts
├── whatsapp-flows-routing.module.ts
├── components/
│   ├── flows-list/               ← Listagem (index.html view #v-listing)
│   │   ├── flows-list.component.ts
│   │   ├── flows-list.component.html
│   │   └── flows-list.component.scss
│   ├── flow-builder/             ← Builder (index.html view #v-builder)
│   │   ├── flow-builder.component.ts
│   │   ├── flow-builder.component.html
│   │   └── flow-builder.component.scss
│   └── flow-preview/             ← Preview do WhatsApp (painel direito)
│       ├── flow-preview.component.ts
│       └── flow-preview.component.html
├── modals/
│   └── publish-flow-modal/       ← Modal de publicação (único modal custom)
│       ├── publish-flow-modal.component.ts
│       └── publish-flow-modal.component.html
├── services/
│   └── whatsapp-flows.service.ts ← Toda a lógica de API
└── models/
    └── whatsapp-flow.model.ts    ← Interfaces TypeScript
```

---

## 10. ROTA SUGERIDA

```typescript
// whatsapp-flows-routing.module.ts
const routes: Routes = [
  {
    path: '',
    component: FlowsListComponent,           // /whatsapp/flows
  },
  {
    path: 'novo',
    component: FlowBuilderComponent,          // /whatsapp/flows/novo
  },
  {
    path: ':id/editar',
    component: FlowBuilderComponent,          // /whatsapp/flows/:id/editar
  },
  {
    path: ':id/visualizar',
    component: FlowBuilderComponent,          // /whatsapp/flows/:id/visualizar
    data: { readOnly: true }
  }
];
```

---

## 11. INTERFACES TYPESCRIPT (Models)

```typescript
// whatsapp-flow.model.ts

export type FlowStatus = 'publicado' | 'rascunho' | 'depreciado' | 'erro';
export type FieldType = 'text' | 'radio' | 'checkbox' | 'dropdown' | 'date';

export interface IWhatsAppFlow {
  id: string | number;
  name: string;
  number: string;
  title: string;
  submitButtonText: string;
  confirmationMessage?: string;
  status: FlowStatus;
  createdAt: string;
  fields: IFlowField[];
}

export interface IFlowField {
  id: string | number;
  type: FieldType;
  label: string;
  required: boolean;
  order: number;
  options?: IFieldOption[];
}

export interface IFieldOption {
  id?: string | number;
  value: string;
  order: number;
}

export interface IFlowsListResponse {
  items: IWhatsAppFlow[];
  totalItems: number;
  totalPages: number;
  currentPage: number;
  pageSize: number;
}

export interface IFlowsListFilter {
  search?: string;
  number?: string;
  status?: FlowStatus;
  sortBy?: 'recentes' | 'antigos' | 'nome';
  page?: number;
  pageSize?: number;
}
```

---

## 12. REFERÊNCIAS RÁPIDAS DE ARQUIVOS

| O que buscar | Arquivo no Angular |
|---|---|
| Todas as cores CSS vars | `src/app/@theme/style-guide/css-files/colors.scss` |
| Tipografia e classes de texto | `src/app/@theme/style-guide/css-files/typography.scss` |
| Tokens de grid, sombras, utilitários | `src/app/@theme/style-guide/css-files/tokens-and-grids.scss` |
| Classes utilitárias (loop fs-*, gap-*, br-*…) | `src/styles.scss` |
| Configuração do tema Material | `src/theme.scss` |
| Componente `<ccm-button>` | `src/app/@shared/components/button/` |
| Modal de confirmação | `src/app/@shared/modals/simple-confirmation-modal/` |
| Autosave indicator | `src/app/@shared/components/autosave-indicator/` |
| Feedback/Snackbar service | `src/app/@shared/services/feedback.service.ts` |
| Snackbar component visual | `src/app/@shared/components/feedback-snackbar/` |
| Paginador de tabela | `src/app/@shared/components/ccm-table-paginator/` |
| Módulo de números WhatsApp (referência) | `src/app/@modules/whatsapp-numbers/` |
| Módulo de templates (referência de estrutura) | `src/app/@modules/templates/` |
