# Guia de Implementação Angular — WhatsApp Flows

> Este documento guia o desenvolvedor Angular na implementação da feature
> **Gerenciador de WhatsApp Flows** no projeto `ngx-ccm-main`.
>
> Leia em conjunto com:
> - `ANGULAR_MAPPING.md` — tabela completa de de-para de tokens e componentes
> - `BDD_cenarios.md` — cenários de negócio (comportamento esperado)
> - `TDD_testes.md` — critérios de aceite e testes
> - `index.html` + `app.css` — protótipo de referência visual com anotações

---

## Pré-requisitos

- Projeto `ngx-ccm-main` configurado e rodando
- Angular 16+
- `@shared/components/button` (standalone) disponível
- `SharedModule` importado no novo módulo
- `MatDialogModule`, `MatTableModule`, `MatFormFieldModule`, `MatSelectModule`,
  `MatSlideToggleModule` já incluídos via `SharedModule` ou Material

---

## Passo 1 — Criar o Feature Module

```bash
ng generate module @modules/whatsapp-flows --routing
```

**`whatsapp-flows.module.ts`** — estrutura mínima:
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ReactiveFormsModule } from '@angular/forms';
import { SharedModule } from '../../@shared/shared.module';
import { ButtonComponent } from '../../@shared/components/button/button.component';
import { WhatsappFlowsRoutingModule } from './whatsapp-flows-routing.module';

// Componentes do módulo
import { FlowsListComponent } from './components/flows-list/flows-list.component';
import { FlowBuilderComponent } from './components/flow-builder/flow-builder.component';
import { FlowPreviewComponent } from './components/flow-preview/flow-preview.component';
import { PublishFlowModalComponent } from './modals/publish-flow-modal/publish-flow-modal.component';

// Angular Material
import { MatTableModule } from '@angular/material/table';
import { MatFormFieldModule } from '@angular/material/form-field';
import { MatInputModule } from '@angular/material/input';
import { MatSelectModule } from '@angular/material/select';
import { MatSlideToggleModule } from '@angular/material/slide-toggle';

@NgModule({
  declarations: [
    FlowsListComponent,
    FlowBuilderComponent,
    FlowPreviewComponent,
    PublishFlowModalComponent,
  ],
  imports: [
    CommonModule,
    ReactiveFormsModule,
    SharedModule,
    ButtonComponent, // standalone
    WhatsappFlowsRoutingModule,
    MatTableModule,
    MatFormFieldModule,
    MatInputModule,
    MatSelectModule,
    MatSlideToggleModule,
  ]
})
export class WhatsappFlowsModule {}
```

---

## Passo 2 — Registrar a Rota no App

Em `src/app/app-routing.module.ts`, adicionar a rota lazy-loaded:
```typescript
{
  path: 'whatsapp/flows', // ou o path definido pelo time
  loadChildren: () =>
    import('./@modules/whatsapp-flows/whatsapp-flows.module')
      .then(m => m.WhatsappFlowsModule),
},
```

Também adicionar ao menu lateral e ao `app-header` via `header-menu.service.ts`, seguindo
o padrão dos outros módulos (ex: `whatsapp-numbers`, `templates`).

---

## Passo 3 — Model e Serviço

### `whatsapp-flow.model.ts`

Ver interfaces completas em `ANGULAR_MAPPING.md` seção 11. Copiar e ajustar
conforme o contrato da API real.

### `whatsapp-flows.service.ts`

```typescript
import { Injectable } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable } from 'rxjs';
import {
  IWhatsAppFlow,
  IFlowsListResponse,
  IFlowsListFilter
} from '../models/whatsapp-flow.model';

@Injectable({ providedIn: 'root' })
export class WhatsappFlowsService {

  private readonly baseUrl = '/api/whatsapp-flows'; // ajustar conforme ambiente

  constructor(private http: HttpClient) {}

  // GET /flows (com filtros e paginação)
  getFlows(filter: IFlowsListFilter): Observable<IFlowsListResponse> {
    let params = new HttpParams();
    if (filter.search) params = params.set('search', filter.search);
    if (filter.number) params = params.set('number', filter.number);
    if (filter.status) params = params.set('status', filter.status);
    if (filter.sortBy) params = params.set('sortBy', filter.sortBy);
    if (filter.page != null) params = params.set('page', filter.page);
    if (filter.pageSize) params = params.set('pageSize', filter.pageSize);
    return this.http.get<IFlowsListResponse>(this.baseUrl, { params });
  }

  // GET /flows/:id
  getFlowById(id: string | number): Observable<IWhatsAppFlow> {
    return this.http.get<IWhatsAppFlow>(`${this.baseUrl}/${id}`);
  }

  // POST /flows
  createFlow(flow: Partial<IWhatsAppFlow>): Observable<IWhatsAppFlow> {
    return this.http.post<IWhatsAppFlow>(this.baseUrl, flow);
  }

  // PATCH /flows/:id
  updateFlow(id: string | number, flow: Partial<IWhatsAppFlow>): Observable<IWhatsAppFlow> {
    return this.http.patch<IWhatsAppFlow>(`${this.baseUrl}/${id}`, flow);
  }

  // DELETE /flows/:id
  deleteFlow(id: string | number): Observable<void> {
    return this.http.delete<void>(`${this.baseUrl}/${id}`);
  }

  // POST /flows/:id/publish
  publishFlow(id: string | number): Observable<IWhatsAppFlow> {
    return this.http.post<IWhatsAppFlow>(`${this.baseUrl}/${id}/publish`, {});
  }

  // POST /flows/:id/deprecate
  deprecateFlow(id: string | number): Observable<IWhatsAppFlow> {
    return this.http.post<IWhatsAppFlow>(`${this.baseUrl}/${id}/deprecate`, {});
  }

  // POST /flows/:id/duplicate
  duplicateFlow(id: string | number): Observable<IWhatsAppFlow> {
    return this.http.post<IWhatsAppFlow>(`${this.baseUrl}/${id}/duplicate`, {});
  }

  // Autosave: PATCH parcial durante edição
  autosaveFlow(id: string | number, partial: Partial<IWhatsAppFlow>): Observable<IWhatsAppFlow> {
    return this.http.patch<IWhatsAppFlow>(`${this.baseUrl}/${id}`, partial);
  }
}
```

---

## Passo 4 — Listagem (`FlowsListComponent`)

### Template Reference: `index.html` → `#v-listing`

```typescript
// flows-list.component.ts
import { Component, OnInit } from '@angular/core';
import { MatDialog } from '@angular/material/dialog';
import { FormControl } from '@angular/forms';
import { debounceTime, distinctUntilChanged } from 'rxjs/operators';

import { WhatsappFlowsService } from '../../services/whatsapp-flows.service';
import { FeedbackService } from '../../../../@shared/services/feedback.service';
import { SimpleConfirmationModalComponent } from '../../../../@shared/modals/simple-confirmation-modal/simple-confirmation-modal.component';
import { IWhatsAppFlow, IFlowsListFilter, FlowStatus } from '../../models/whatsapp-flow.model';
import { CCMPaginatorOutput } from '../../../../@shared/models/ccm-table-paginator.model';

@Component({
  selector: 'app-flows-list',
  templateUrl: './flows-list.component.html',
  styleUrls: ['./flows-list.component.scss']
})
export class FlowsListComponent implements OnInit {

  flows: IWhatsAppFlow[] = [];
  totalItems = 0;
  totalPages = 0;
  currentPage = 0;
  pageSize = 10;
  isLoading = false;

  // Filtros — equivalentes a #search, #filterNum, #filterStatus, #sortBy no protótipo
  searchControl = new FormControl('');
  filterNumber = '';
  filterStatus = '';
  sortBy: 'recentes' | 'antigos' | 'nome' = 'recentes';

  displayedColumns = ['name', 'number', 'status', 'createdAt', 'actions'];

  constructor(
    private flowsService: WhatsappFlowsService,
    private dialog: MatDialog,
    private feedback: FeedbackService,
  ) {}

  ngOnInit(): void {
    this.loadFlows();

    // Busca com debounce (equivalente ao oninput="renderListing()" do protótipo)
    this.searchControl.valueChanges.pipe(
      debounceTime(300),
      distinctUntilChanged()
    ).subscribe(() => {
      this.currentPage = 0;
      this.loadFlows();
    });
  }

  loadFlows(): void {
    this.isLoading = true;
    const filter: IFlowsListFilter = {
      search: this.searchControl.value || undefined,
      number: this.filterNumber || undefined,
      status: (this.filterStatus as FlowStatus) || undefined,
      sortBy: this.sortBy,
      page: this.currentPage,
      pageSize: this.pageSize,
    };

    this.flowsService.getFlows(filter).subscribe({
      next: (res) => {
        this.flows = res.items;
        this.totalItems = res.totalItems;
        this.totalPages = res.totalPages;
        this.isLoading = false;
      },
      error: () => {
        this.feedback.error('Erro ao carregar os flows.');
        this.isLoading = false;
      }
    });
  }

  onPageChange(event: CCMPaginatorOutput): void {
    this.currentPage = event.page;
    this.pageSize = event.pageSize;
    this.loadFlows();
  }

  // Ações por status (equivalente a buildActions() no protótipo)
  canEdit(flow: IWhatsAppFlow): boolean {
    return flow.status === 'rascunho' || flow.status === 'erro';
  }
  canView(flow: IWhatsAppFlow): boolean {
    return flow.status === 'publicado';
  }
  canDeprecate(flow: IWhatsAppFlow): boolean {
    return flow.status === 'publicado';
  }
  canDuplicate(flow: IWhatsAppFlow): boolean {
    return flow.status !== 'depreciado';
  }
  canDelete(flow: IWhatsAppFlow): boolean {
    return flow.status === 'rascunho' || flow.status === 'erro';
  }

  // Depreciar (equivalente a openDeprecate() no protótipo)
  openDeprecate(flow: IWhatsAppFlow): void {
    this.dialog.open(SimpleConfirmationModalComponent, {
      data: {
        title: 'Depreciar flow',
        content: `Você está prestes a depreciar o flow "<strong>${flow.name}</strong>".<br><br>
                  Este flow está sendo usado em templates e jornadas ativas.
                  Depreciá-lo impedirá o envio nesses contextos.`,
        modalWidth: '460px',
        showHeaderCloseBtn: true,
        buttonConfig: {
          cancelLabel: 'Cancelar',
          cancelBtnType: 'terciary',
          confirmLabel: 'Depreciar',
        }
      }
    }).afterClosed().subscribe(result => {
      if (result?.success) {
        this.flowsService.deprecateFlow(flow.id).subscribe({
          next: () => {
            this.feedback.success(`Flow "${flow.name}" depreciado`);
            this.loadFlows();
          },
          error: () => this.feedback.error('Erro ao depreciar o flow.')
        });
      }
    });
  }

  // Excluir (equivalente a openDelete() no protótipo)
  openDelete(flow: IWhatsAppFlow): void {
    this.dialog.open(SimpleConfirmationModalComponent, {
      data: {
        title: 'Excluir flow',
        content: `Tem certeza que deseja excluir permanentemente o flow
                  "<strong>${flow.name}</strong>"? Esta ação não pode ser desfeita.`,
        modalWidth: '460px',
        showHeaderCloseBtn: true,
        buttonConfig: {
          cancelLabel: 'Cancelar',
          cancelBtnType: 'terciary',
          confirmLabel: 'Excluir',
        }
      }
    }).afterClosed().subscribe(result => {
      if (result?.success) {
        this.flowsService.deleteFlow(flow.id).subscribe({
          next: () => {
            this.feedback.success('Flow excluído');
            this.loadFlows();
          },
          error: () => this.feedback.error('Erro ao excluir o flow.')
        });
      }
    });
  }

  // Duplicar
  duplicateFlow(flow: IWhatsAppFlow): void {
    this.flowsService.duplicateFlow(flow.id).subscribe({
      next: () => {
        this.feedback.success(`"${flow.name}" duplicado como rascunho`);
        this.loadFlows();
      },
      error: () => this.feedback.error('Erro ao duplicar o flow.')
    });
  }
}
```

---

## Passo 5 — Builder (`FlowBuilderComponent`)

### Template Reference: `index.html` → `#v-builder`

```typescript
// flow-builder.component.ts
import { Component, OnInit, OnDestroy } from '@angular/core';
import { FormBuilder, FormGroup, FormArray, Validators } from '@angular/forms';
import { ActivatedRoute, Router } from '@angular/router';
import { MatDialog } from '@angular/material/dialog';
import { Subject } from 'rxjs';
import { debounceTime, takeUntil } from 'rxjs/operators';

import { WhatsappFlowsService } from '../../services/whatsapp-flows.service';
import { FeedbackService } from '../../../../@shared/services/feedback.service';
import { PublishFlowModalComponent } from '../../modals/publish-flow-modal/publish-flow-modal.component';
import { FieldType } from '../../models/whatsapp-flow.model';

@Component({
  selector: 'app-flow-builder',
  templateUrl: './flow-builder.component.html',
  styleUrls: ['./flow-builder.component.scss']
})
export class FlowBuilderComponent implements OnInit, OnDestroy {

  form!: FormGroup;
  flowId?: string;
  isReadOnly = false;
  isSaving = false;            // Controla <ccm-autosave-indicator [loading]>
  hasMinRequirements = false;  // Controla <ccm-autosave-indicator [disabledByMinRequirements]>
  isPublishing = false;

  private destroy$ = new Subject<void>();

  readonly fieldTypes: { type: FieldType; label: string; icon: string }[] = [
    { type: 'text',     label: 'Texto livre',      icon: 'ph-text-aa' },
    { type: 'radio',    label: 'Seleção única',     icon: 'ph-radio-button' },
    { type: 'checkbox', label: 'Múltipla escolha',  icon: 'ph-check-square' },
    { type: 'dropdown', label: 'Dropdown',           icon: 'ph-caret-circle-down' },
    { type: 'date',     label: 'Data',               icon: 'ph-calendar' },
  ];

  readonly numbers = ['11 3777-1887', '11 3777-9335', '11 3777-7627']; // via API

  constructor(
    private fb: FormBuilder,
    private route: ActivatedRoute,
    private router: Router,
    private flowsService: WhatsappFlowsService,
    private feedback: FeedbackService,
    private dialog: MatDialog,
  ) {}

  ngOnInit(): void {
    this.flowId = this.route.snapshot.paramMap.get('id') || undefined;
    this.isReadOnly = !!this.route.snapshot.data['readOnly'];

    this.buildForm();

    if (this.flowId) {
      this.loadFlow(this.flowId);
    }

    // Autosave — equivalente a autosaveNow() com debounce no protótipo
    if (!this.isReadOnly) {
      this.form.valueChanges.pipe(
        debounceTime(900),
        takeUntil(this.destroy$)
      ).subscribe(() => {
        if (this.flowId && this.form.dirty) {
          this.autosave();
        }
        this.hasMinRequirements = this.form.get('fields')!.value.length > 0
          && !!this.form.get('name')!.value;
      });
    }
  }

  buildForm(): void {
    this.form = this.fb.group({
      name:                ['', Validators.required],
      number:              ['', Validators.required],
      title:               ['', Validators.required],
      submitButtonText:    ['', Validators.required],
      confirmationMessage: [''],
      fields:              this.fb.array([]),
    });
  }

  loadFlow(id: string): void {
    this.flowsService.getFlowById(id).subscribe({
      next: (flow) => {
        this.form.patchValue({
          name:                flow.name,
          number:              flow.number,
          title:               flow.title,
          submitButtonText:    flow.submitButtonText,
          confirmationMessage: flow.confirmationMessage || '',
        });
        flow.fields.forEach(f => this.addFieldFromData(f));
        if (this.isReadOnly) this.form.disable();
      },
      error: () => this.feedback.error('Erro ao carregar o flow.')
    });
  }

  get fields(): FormArray {
    return this.form.get('fields') as FormArray;
  }

  // Adicionar campo (equivalente a addField(type) no protótipo)
  addField(type: FieldType): void {
    const hasOptions = ['radio', 'checkbox', 'dropdown'].includes(type);
    const fieldGroup = this.fb.group({
      type:     [type],
      label:    ['', Validators.required],
      required: [true],
      options:  this.fb.array(
        hasOptions ? [this.fb.control(''), this.fb.control('')] : []
      ),
    });
    this.fields.push(fieldGroup);
  }

  addFieldFromData(data: any): void {
    const fieldGroup = this.fb.group({
      type:     [data.type],
      label:    [data.label, Validators.required],
      required: [data.required],
      options:  this.fb.array((data.options || []).map((o: any) => this.fb.control(o.value))),
    });
    this.fields.push(fieldGroup);
  }

  removeField(index: number): void {
    this.fields.removeAt(index);
  }

  moveField(index: number, direction: -1 | 1): void {
    const newIndex = index + direction;
    if (newIndex < 0 || newIndex >= this.fields.length) return;
    const field = this.fields.at(index);
    this.fields.removeAt(index);
    this.fields.insert(newIndex, field);
  }

  getOptions(fieldIndex: number): FormArray {
    return this.fields.at(fieldIndex).get('options') as FormArray;
  }

  addOption(fieldIndex: number): void {
    this.getOptions(fieldIndex).push(this.fb.control(''));
  }

  removeOption(fieldIndex: number, optionIndex: number): void {
    const opts = this.getOptions(fieldIndex);
    if (opts.length <= 2) {
      this.feedback.warn('Mínimo de 2 opções para este tipo de campo');
      return;
    }
    opts.removeAt(optionIndex);
  }

  autosave(): void {
    if (!this.flowId) return;
    this.isSaving = true;
    this.flowsService.autosaveFlow(this.flowId, this.form.value).subscribe({
      next: () => { this.isSaving = false; this.form.markAsPristine(); },
      error: () => { this.isSaving = false; }
    });
  }

  saveDraft(): void {
    this.autosave();
    this.feedback.success('Rascunho salvo');
  }

  // Publicar (equivalente a openPublish() no protótipo)
  openPublish(): void {
    if (this.form.invalid || this.fields.length === 0) {
      this.feedback.error('Preencha todos os campos obrigatórios antes de publicar.');
      return;
    }

    this.dialog.open(PublishFlowModalComponent, {
      data: {
        flowName:     this.form.get('name')!.value,
        fieldsCount:  this.fields.length,
        number:       this.form.get('number')!.value,
      }
    }).afterClosed().subscribe(confirmed => {
      if (confirmed && this.flowId) {
        this.isPublishing = true;
        this.flowsService.publishFlow(this.flowId).subscribe({
          next: () => {
            this.feedback.success(`Flow "${this.form.get('name')!.value}" publicado com sucesso!`);
            this.router.navigate(['../..'], { relativeTo: this.route });
          },
          error: () => {
            this.feedback.error('Erro ao publicar o flow.');
            this.isPublishing = false;
          }
        });
      }
    });
  }

  goBack(): void {
    this.router.navigate(['../..'], { relativeTo: this.route });
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

---

## Passo 6 — Template HTML do Builder

Referência visual: `index.html` bloco `#v-builder` com anotações `<!-- ANGULAR: ... -->`

```html
<!-- flow-builder.component.html -->

<!-- Breadcrumb + Ações (equivalente a .builder-lhdr) -->
<div class="d-flex align-items-center justify-content-between py-14 px-24
            border-bottom-high-dark-1 bg-white">

  <!-- Breadcrumb com botão voltar -->
  <!-- ANGULAR: back-btn → <ccm-button mini type="quinary"> -->
  <div class="d-flex align-items-center gap-8">
    <ccm-button mini type="quinary" (ccmClick)="goBack()">
      <i class="ph ph-arrow-left"></i>
    </ccm-button>
    <!-- .bc-item e .bc-cur são apenas texto visual, sem comportamento -->
    <span class="body-xx-small-regular color-low-light">Flows</span>
    <i class="ph ph-caret-right fs-11 color-high-dark"></i>
    <span class="body-xx-small-medium">{{ form.get('name')?.value || 'Novo flow' }}</span>
  </div>

  <!-- Autosave indicator -->
  <!-- ANGULAR: div#autosave → <ccm-autosave-indicator> -->
  <ccm-autosave-indicator
    [loading]="isSaving"
    [disabledByMinRequirements]="!hasMinRequirements"
    warningTooltipText="Adicione ao menos um campo para ativar o salvamento automático.">
  </ccm-autosave-indicator>

  <!-- Botões de ação -->
  <!-- ANGULAR: .builder-actions → d-flex gap-8 -->
  <div class="d-flex align-items-center gap-8" *ngIf="!isReadOnly">
    <!-- .btn-quat → type="quaternary" -->
    <ccm-button type="quaternary" medium (ccmClick)="goBack()">
      <p class="body-xx-small-semi-bold">Cancelar</p>
    </ccm-button>
    <!-- .btn-sec → type="secondary" -->
    <ccm-button type="secondary" medium (ccmClick)="saveDraft()">
      <i class="ph ph-floppy-disk"></i>
      <p class="body-xx-small-semi-bold">Salvar rascunho</p>
    </ccm-button>
    <!-- .btn-prim → type="primary" -->
    <ccm-button type="primary" medium [disabled]="isPublishing" (ccmClick)="openPublish()">
      <i class="ph ph-rocket-launch"></i>
      <p class="body-xx-small-semi-bold">Publicar flow</p>
    </ccm-button>
  </div>
</div>

<!-- Corpo do builder -->
<div class="d-flex flex-row" style="flex: 1; overflow: hidden;">

  <!-- Painel de edição (equivalente a .editor-panel) -->
  <div class="d-flex flex-column gap-20 p-24"
       style="flex: 1; overflow-y: auto; border-right: 1px solid var(--neutral-color-high-medium);">

    <form [formGroup]="form">

      <!-- Seção: Configurações gerais (.section) -->
      <!-- ANGULAR: .section → .white-box-background -->
      <div class="white-box-background">
        <div class="d-flex align-items-center px-20 py-14
                    border-bottom-high-light-1">
          <!-- .section-hdr h4 → <h4> -->
          <h4>
            <i class="ph ph-sliders mr-6 color-low-light"></i>
            Configurações gerais
          </h4>
        </div>
        <div class="d-flex flex-column gap-16 p-20">

          <!-- Nome do flow — .ipt → <mat-form-field><input matInput> -->
          <mat-form-field appearance="outline" class="w-100">
            <mat-label>Nome do flow <span class="color-warning-pure">*</span></mat-label>
            <input matInput formControlName="name" placeholder="Ex.: coleta_dados_syscob">
          </mat-form-field>

          <!-- Linha 2: Número + Título -->
          <div class="d-flex gap-14">
            <!-- .sel-wrap → <mat-form-field><mat-select> -->
            <mat-form-field appearance="outline" style="flex: 1">
              <mat-label>Número do WhatsApp <span class="color-warning-pure">*</span></mat-label>
              <mat-select formControlName="number">
                <mat-option value="">Selecione</mat-option>
                <mat-option *ngFor="let n of numbers" [value]="n">{{n}}</mat-option>
              </mat-select>
            </mat-form-field>

            <mat-form-field appearance="outline" style="flex: 1">
              <mat-label>Título da tela <span class="color-warning-pure">*</span></mat-label>
              <input matInput formControlName="title" placeholder="Ex.: Informe seus dados">
            </mat-form-field>
          </div>

          <!-- Linha 3: Texto botão + Confirmação -->
          <div class="d-flex gap-14">
            <mat-form-field appearance="outline" style="flex: 1">
              <mat-label>Texto do botão de envio <span class="color-warning-pure">*</span></mat-label>
              <input matInput formControlName="submitButtonText" placeholder="Ex.: Enviar">
            </mat-form-field>

            <mat-form-field appearance="outline" style="flex: 1">
              <mat-label>Mensagem de confirmação</mat-label>
              <input matInput formControlName="confirmationMessage"
                     placeholder="Ex.: Dados enviados com sucesso!">
            </mat-form-field>
          </div>
        </div>
      </div>

      <!-- Seção: Campos da tela -->
      <div class="white-box-background mt-20">
        <div class="d-flex align-items-center justify-content-between px-20 py-14
                    border-bottom-high-light-1">
          <h4>
            <i class="ph ph-list-bullets mr-6 color-low-light"></i>
            Campos da tela
          </h4>
          <span class="caption-xxxx-small color-low-light">
            {{fields.length}} campo{{fields.length !== 1 ? 's' : ''}}
          </span>
        </div>

        <div class="d-flex flex-column gap-16 p-20" formArrayName="fields">

          <!-- Banner vazio (equivalente a #noFieldsBanner) -->
          <div *ngIf="fields.length === 0"
               class="blue-box-background d-flex align-items-center gap-8 p-12">
            <i class="ph ph-info color-secondary-pure"></i>
            <p class="caption-xxxx-small">
              Adicione ao menos um campo à tela antes de publicar o flow.
            </p>
          </div>

          <!-- Cards de campo (equivalente a .field-card) -->
          <ng-container *ngFor="let field of fields.controls; let i = index" [formGroupName]="i">
            <!-- campo expandível — usar mat-expansion-panel ou custom -->
          </ng-container>

        </div>

        <!-- Botão adicionar campo -->
        <div class="px-20 pb-20">
          <!-- .add-field-btn → botão dashed custom ou ccm-button quinary -->
          <button type="button" class="add-field-btn w-100" [matMenuTriggerFor]="fieldTypePicker">
            <i class="ph ph-plus-circle"></i>
            Adicionar campo
          </button>

          <!-- Menu de tipos de campo (equivalente a .field-picker-inline) -->
          <mat-menu #fieldTypePicker="matMenu">
            <button mat-menu-item *ngFor="let ft of fieldTypes" (click)="addField(ft.type)">
              <i class="ph {{ft.icon}} mr-8"></i>
              <span>{{ft.label}}</span>
            </button>
          </mat-menu>
        </div>
      </div>

    </form>
  </div>

  <!-- Painel de prévia (.preview-panel) -->
  <app-flow-preview
    [title]="form.get('title')?.value"
    [submitLabel]="form.get('submitButtonText')?.value"
    [flowName]="form.get('name')?.value"
    [fields]="fields.value">
  </app-flow-preview>

</div>
```

---

## Passo 7 — Badge de Status

Usar classes do design system Angular. Criar um pipe ou componente:

```typescript
// status-badge.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';
import { FlowStatus } from '../models/whatsapp-flow.model';

@Pipe({ name: 'flowStatusClass' })
export class FlowStatusClassPipe implements PipeTransform {
  transform(status: FlowStatus): string {
    const map: Record<FlowStatus, string> = {
      publicado:  'green-box-background',
      rascunho:   'white-box-background',
      depreciado: 'yellow-box-background',
      erro:       'red-box-background',
    };
    return map[status] || '';
  }
}
```

```html
<!-- Na tabela: -->
<span class="badge caption-xxx-small-semi-bold br-26 px-10 py-3"
      [ngClass]="flow.status | flowStatusClass">
  {{ flow.status | titlecase }}
</span>
```

---

## Passo 8 — Variáveis de Ambiente / Configuração

Verificar em qual `environment.ts` a URL base da API deve ser configurada:
```typescript
// src/environments/environment.ts
export const environment = {
  production: false,
  apiUrl: 'https://api.seudominio.com', // existente
  whatsappFlowsEndpoint: '/whatsapp-flows' // novo endpoint
};
```

---

## Checklist de Implementação

```
[ ] Criar WhatsappFlowsModule com routing
[ ] Registrar rota lazy-loaded no app-routing
[ ] Adicionar item ao menu (header-menu.service)
[ ] Criar WhatsappFlowsService (CRUD + ações)
[ ] Criar IWhatsAppFlow models
[ ] Implementar FlowsListComponent (tabela + filtros + paginação)
[ ] Implementar FlowBuilderComponent (formulário reativo + autosave)
[ ] Implementar FlowPreviewComponent (prévia do WhatsApp)
[ ] Implementar PublishFlowModalComponent
[ ] Substituir modais de delete/deprecate por SimpleConfirmationModalComponent
[ ] Substituir toasts por FeedbackService
[ ] Substituir autosave indicator por <ccm-autosave-indicator>
[ ] Substituir botões por <ccm-button type="...">
[ ] Substituir paginação por <ccm-table-paginator>
[ ] Criar pipe FlowStatusClassPipe para badges
[ ] Testes unitários (ver TDD_testes.md)
[ ] Revisão de acessibilidade (aria-labels)
[ ] Verificar permissões via PermissionsService
```

---

## Referências

- `index.html` — Protótipo HTML com anotações `<!-- ANGULAR: -->` em cada elemento
- `app.css` — Protótipo CSS com comentários `/* Angular: */` em cada token
- `ANGULAR_MAPPING.md` — Tabela completa de de-para
- `BDD_cenarios.md` — Comportamentos esperados
- `TDD_testes.md` — Critérios de aceite
- `src/app/@shared/` — Todos os componentes reutilizáveis do ngx-ccm
- `src/app/@modules/whatsapp-numbers/` — Módulo de referência similar
- `src/app/@theme/style-guide/` — Design tokens completos
