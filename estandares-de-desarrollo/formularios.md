# Implementación de formularios en Angular

## 1. **Creación de formularios**

Para la creación de formularios en Angular se debe utilizar `FormGroup`, `FormControl` y `FormArray` de la librería `@angular/forms`.

```typescript
export class Component {

    formControl = new FormControl<string>('', {nonNullable: true});
    formGroup = new FormGroup({
        name: new FormControl<string>('', {nonNullable: true}),
        age: new FormControl<number | null>(null),
        email: new FormControl<string | null>(null),
    });
    formArray = new FormArray<FormControl<string>>([]);

}
```
## 2. **Integración con el template**

Para enlazar los formularios se debe importar el módulo `ReactiveFormsModule` en el módulo correspondiente.

```typescript
import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
  imports: [
    ReactiveFormsModule
  ]
})
export class AppModule { }
```

Esto no permitira usar la directiva `formControl` que es la que principalmente se utiliza para enlazar los formularios en el template.

```html
<form>
    <input [formControl]="formGroup.controls.name" type="text" placeholder="Name">
    <input [formControl]="formGroup.controls.age" type="number" placeholder="Age">
    <input [formControl]="formGroup.controls.email" type="email" placeholder="Email">
</form>
```