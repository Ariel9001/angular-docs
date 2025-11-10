# Guía de uso: `ListService` y `PaginatedListService`

Servicios de la librería `@cbm-common/data-access` para consumir listas de datos con señales reactivas.

---

## Cuándo usar cada servicio

| Servicio | Uso |
|----------|-----|
| **ListService** | Listas sin paginación, ng-select, catálogos pequeños |
| **PaginatedListService** | Tablas con paginación, listas grandes (>50 items) |

---

## 1. ListService

### Configuración básica

```typescript
constructor(
  private readonly destroyRef: DestroyRef,
  @Inject(MY_REPOSITORY) private readonly repository: MyRepository
) {}

myService = new ListService(
  {
    destroyRef: this.destroyRef,
    list$: this.repository.list.bind(this.repository)
  },
  {
    mapFn: (res) => res.data || []  // Extrae los datos de la respuesta
  }
);

ngOnInit() {
  this.myService.list();  // Cargar datos
}
```

### Uso en ng-select (sin búsqueda)

```typescript
// Componente
messageSettings$ = new ListService(
  {
    destroyRef: this.destroyRef,
    list$: messageSettingsRepository.list.bind(messageSettingsRepository)
  },
  {
    mapFn: (res) => res.data || []
  }
);

ngOnInit() {
  this.messageSettings$.list();  // Cargar datos
}

```

```html
<!-- Template -->
<ng-select
  [items]="messageSettings$.data()"
  [loading]="messageSettings$.loading()"
  placeholder="Seleccione un motivo"
  [formControl]="form.controls.reason"
>
  <ng-template ng-label-tmp ng-option-tmp let-item="item">
    <span class="badge">{{ item.description_process }}</span>
    {{ item.description }}
  </ng-template>
</ng-select>
```

### Uso en ng-select (con búsqueda typeahead)

```typescript
// Componente
constructor(
  private readonly destroyRef: DestroyRef,
  @Inject(USER_REPOSITORY) private readonly userRepository: UserRepository
) {}

users$ = new ListService(
  {
    destroyRef: this.destroyRef,
    list$: this.userRepository.search.bind(this.userRepository)
  },
  {
    typeahead: 'name',  // Campo para búsqueda
    delay: 300          // Debounce en ms
  }
);

ngOnInit() {
  this.users$.list();
}
```

```html
<!-- Template -->
<ng-select
  [items]="users$.data()"
  [loading]="users$.loading()"
  [typeahead]="users$.typeahead$"
  placeholder="Buscar usuario..."
  [formControl]="userControl"
>
  <ng-template ng-option-tmp let-item="item">
    {{ item.name }} - {{ item.email }}
  </ng-template>
</ng-select>
```

### Métodos principales

```typescript
// Gestión de parámetros
this.service.setParams({ status: 'active' });           // Reemplaza parámetros
this.service.updateParams({ category: 'electronics' }); // Actualiza parcialmente
this.service.getParams();                                // Obtiene parámetros actuales

// Carga de datos
this.service.list();  // Cargar con parámetros actuales

// Búsqueda typeahead
this.service.typeahead$.next('search term');
```

---

## 2. PaginatedListService

### Configuración básica

```typescript
constructor(
  private readonly destroyRef: DestroyRef,
  @Inject(DOWN_PAYMENT_REPOSITORY) 
  private readonly downPaymentRepository: DownPaymentDomainRepository,
  @Inject(NOTIFICATION_SERVICE)
  private readonly notificationService: CbmNotificationService
) {}

downPayment$ = new PaginatedListService(
  {
    destroyRef: this.destroyRef,
    list$: this.downPaymentRepository.list.bind(this.downPaymentRepository)
  },
  {
    mapFn: (res) => res.items,
    notificationService: this.notificationService,
    size: 10  // Registros por página
  }
);

ngOnInit() {
  this.downPayment$.list();
}
```

### Uso en tabla con paginación

```typescript
// Componente
products$ = new PaginatedListService(
  {
    destroyRef: this.destroyRef,
    list$: productsRepository.list.bind(productsRepository)
  },
  {
    size: 10,
    synchronize_pagination: true,  // Actualiza desde la respuesta del servidor
    mapFn: (res) => res.data;
  }
);
```

```html
<!-- Template -->
<table class="table">
  <thead>
    <tr>
      <th>N°</th>
      <th>Fecha</th>
      <th>Documento</th>
      <th>Total</th>
    </tr>
  </thead>
  <tbody>
    @if (products$.loading()) {
      <tr><td colspan="4">Cargando...</td></tr>
    } @else {
      @for (item of products$.data(); track $index) {
        <tr>
          <td>{{ $index + 1 }}</td>
          <td>{{ item.date | date: "dd/MM/yyyy" }}</td>
          <td>{{ item.document_number }}</td>
          <td>{{ item.total | number: "1.2-2" }}</td>
        </tr>
      } @empty {
        <tr><td colspan="4">Sin registros</td></tr>
      }
    }
  </tbody>
</table>

<app-pagination
  [signal-pagination]="products$.signalPaginated"
  (change-signal)="products$.list(false)"
  [current]="products$.signalPaginated().page"
  [total]="products$.signalPaginated().pages"
/>
```

### Métodos principales

```typescript
// Navegación
this.service.nextPage(null);  // Siguiente página

// Gestión de paginación
this.service.getPaginated();                    // Obtiene estado actual
this.service.updatePaginated({ page: 3 });     // Actualiza página
this.service.setPaginated({ page: 1, size: 20, pages: 5, records: 100 });

// Señal reactiva
this.service.signalPaginated()  // { page, size, pages, records }

// Validar navegación
const pg = this.service.getPaginated();
const canGoNext = pg.page < pg.pages;
const canGoPrev = pg.page > 1;
```
---

## Configuración completa

### ListService

```typescript
interface ListModel.Config<T, R = T> {
  mapFn?: (res: T) => R;
  delay?: number;                            // Default: 600ms
  notificationService?: CbmNotificationService;
  typeahead?: string;                        // Campo para búsqueda
}
```

### PaginatedListService

```typescript
interface PaginatedListModel.Config<T, R = T> {
  mapFn?: (res: T, pg: WritableSignal<Paginated>) => R;
  typeahead?: string;
  synchronize_pagination?: boolean;  // Sincroniza desde servidor
  merge_results?: boolean;           // Acumula resultados (scroll infinito)
  size?: number;                     // Default: 10
  notificationService?: CbmNotificationService;
  delay?: number;
}
```

---

## Reglas importantes

### ✅ Hacer

```typescript
// 1. Siempre inyectar DestroyRef
constructor(
 private readonly destroyRef: DestroyRef
) { }

// 2. Llamar a list() después de setParams()
this.service.setParams({ filter: 'active' });
this.service.list();

// 3. Usar mapFn para extraer datos
mapFn: (res) => res.data || []

// 4. Configurar notificationService para ver errores
{ notificationService: this.notificationService }
```

### ❌ Evitar

```typescript
// 1. No configurar parámetros sin llamar list()
this.service.setParams({ filter: 'active' });  // ❌ Falta list()

// 2. No manipular directamente data() o loading()
this.service.data.set([]);  // ❌ Son readonly

// 3. No mezclar page/size con setParams en PaginatedListService
this.service.setParams({ page: 2, size: 20 });  // ❌ Usa updatePaginated()
```

---

## Errores comunes y soluciones

| Problema | Solución |
|----------|----------|
| Lista no se actualiza | Llamar a `list()` después de `setParams()` |
| Typeahead no funciona | Configurar `{ typeahead: 'campo' }` y emitir con `typeahead$.next()` |
| Paginación no sincroniza | Activar `synchronize_pagination: true` y actualizar en `mapFn` |
| Errores no se muestran | Pasar `notificationService` en config unas vez injectado en el constructor |
