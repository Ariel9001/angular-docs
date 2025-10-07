# Uso y creación de modelos

## 1. **Creación de modelos**

Para la creación de modelos principalmente se requiere de un objeto para poder definir correctamente las interfaces

```typescript
{
    name: "...",
    edad: 18,
    parents: {
        father: "...",
        mother: "..."
    },
    hobbies: ["...", "..."]
    brothers: [
        {
            name: "...",
            edad: 18
        },
        {
            name: "...",
            edad: 20
        }
    ]
}
```

Con base en el objeto anterior se puede definir la interfaz tomando en cuenta el uso de [comentarios](./comentarios.md)

- Declarar un `namespace` para agrupar los modelos relacionados el prefijo `I` para identificar que es una interfaz.
- Si la estructura del objeto es compleja, se recomienda crear `namespaces` adicionales para definir las estructuras anidadas.

```typescript
export namespace IModels {
  //#region level1
  export interface level1 {
    name: string;
    edad: number;
    parents: level1.Parents;
    hobbies: string[];
    brothers: level1.IBrother[];
  }

  export namespace level1 {
    export interface Parents {
      father: string;
      mother: string;
    }

    export interface IBrother {
      name: string;
      edad: number;
    }
  }
}
```
