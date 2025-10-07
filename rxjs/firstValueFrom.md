# Uso `firstValueFrom` para consumir Observables

## Cuando implementar `firstValueFrom`

En Angular, es muy común trabajar con Observables, por lo que es importante saber como manejar funciones donde se usen Observables. En este caso, se recomienda usar `firstValueFrom` para convertir un Observable en una Promesa y así poder usar `async/await` para manejar la asincronía de manera más sencilla y legible.

## 1. **Implementación de firstValueFrom**

- En varios casos es necesario convertir un Observable a una Promesa. Para esto, utiliza `firstValueFrom` de `rxjs`.
- Evita usar `.toPromise()` no lo uses.
- Siempre maneja errores al usar `firstValueFrom` con un bloque try-catch.
  Ej:

```typescript
import { firstValueFrom } from "rxjs";

export class Service {
  constructor(
    private readonly repository: Repository,
    private readonly destroyRef: DestroyRef
  ) {}

  async fetchData(): Promise<DataType> {
    try {
      const get$ = this.repository
        .getData()
        .pipe(takeUntilDestroyed(this.destroyRef));
      const { data } = await firstValueFrom(get$);
      return data;
    } catch (error) {
      console.error("Error fetching data:", error);
      throw error;
    }
  }
}
```
