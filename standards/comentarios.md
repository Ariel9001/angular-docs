# Comentarios en el Código

## 1. **Uso de `#region`**

- Utiliza `#region` para agrupar secciones de variables como signals, computeds, variables de estado, etc.
- Los comentarios de `#region` siempre deben estar en inglés.

```typescript
// #region Signals
const data = signal<DataType | null>(null);

//#region state
const isLoading = signal(false);

//#region Computeds
const isDataAvailable = computed(() => !!data());
```
