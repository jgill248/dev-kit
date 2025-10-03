# Angular Reactive Forms

## Description
Guide for creating reactive forms in Angular with validation, dynamic controls, and best practices using FormBuilder and FormGroup.

## Style Rules
- Use ReactiveFormsModule for form handling
- Implement FormBuilder for cleaner form creation
- Add both synchronous and asynchronous validators
- Use FormGroup and FormControl with proper typing
- Implement proper error handling and display
- Handle form submission with proper validation
- Use OnPush change detection when possible
- Unsubscribe from observables in ngOnDestroy

## Example: Basic Reactive Form

```typescript
// user-form.component.ts
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup, Validators } from '@angular/forms';
import { UserService } from '../services/user.service';

@Component({
  selector: 'app-user-form',
  templateUrl: './user-form.component.html',
  styleUrls: ['./user-form.component.css'],
})
export class UserFormComponent implements OnInit {
  userForm: FormGroup;
  isSubmitting = false;

  constructor(
    private fb: FormBuilder,
    private userService: UserService
  ) {
    this.userForm = this.fb.group({
      name: ['', [Validators.required, Validators.minLength(2)]],
      email: ['', [Validators.required, Validators.email]],
      phone: ['', [Validators.pattern(/^\+?[1-9]\d{1,14}$/)]],
      age: ['', [Validators.required, Validators.min(18), Validators.max(120)]],
      address: this.fb.group({
        street: ['', Validators.required],
        city: ['', Validators.required],
        zipCode: ['', [Validators.required, Validators.pattern(/^\d{5}$/)]],
      }),
    });
  }

  ngOnInit(): void {
    // Load initial data if editing
    // this.loadUserData();
  }

  get name() {
    return this.userForm.get('name');
  }

  get email() {
    return this.userForm.get('email');
  }

  get age() {
    return this.userForm.get('age');
  }

  get address() {
    return this.userForm.get('address') as FormGroup;
  }

  onSubmit(): void {
    if (this.userForm.valid) {
      this.isSubmitting = true;
      this.userService.createUser(this.userForm.value).subscribe({
        next: (response) => {
          console.log('User created:', response);
          this.userForm.reset();
          this.isSubmitting = false;
        },
        error: (error) => {
          console.error('Error creating user:', error);
          this.isSubmitting = false;
        },
      });
    } else {
      this.markFormGroupTouched(this.userForm);
    }
  }

  private markFormGroupTouched(formGroup: FormGroup): void {
    Object.keys(formGroup.controls).forEach(key => {
      const control = formGroup.get(key);
      control?.markAsTouched();

      if (control instanceof FormGroup) {
        this.markFormGroupTouched(control);
      }
    });
  }

  getErrorMessage(controlName: string): string {
    const control = this.userForm.get(controlName);
    if (control?.hasError('required')) {
      return 'This field is required';
    }
    if (control?.hasError('email')) {
      return 'Please enter a valid email';
    }
    if (control?.hasError('minlength')) {
      return `Minimum length is ${control.errors?.['minlength'].requiredLength}`;
    }
    if (control?.hasError('min')) {
      return `Minimum value is ${control.errors?.['min'].min}`;
    }
    if (control?.hasError('max')) {
      return `Maximum value is ${control.errors?.['max'].max}`;
    }
    if (control?.hasError('pattern')) {
      return 'Invalid format';
    }
    return '';
  }
}
```

## Example: Template

```html
<!-- user-form.component.html -->
<form [formGroup]="userForm" (ngSubmit)="onSubmit()">
  <div class="form-group">
    <label for="name">Name *</label>
    <input
      id="name"
      type="text"
      formControlName="name"
      class="form-control"
      [class.is-invalid]="name?.invalid && name?.touched"
    />
    <div class="invalid-feedback" *ngIf="name?.invalid && name?.touched">
      {{ getErrorMessage('name') }}
    </div>
  </div>

  <div class="form-group">
    <label for="email">Email *</label>
    <input
      id="email"
      type="email"
      formControlName="email"
      class="form-control"
      [class.is-invalid]="email?.invalid && email?.touched"
    />
    <div class="invalid-feedback" *ngIf="email?.invalid && email?.touched">
      {{ getErrorMessage('email') }}
    </div>
  </div>

  <div class="form-group">
    <label for="phone">Phone</label>
    <input
      id="phone"
      type="tel"
      formControlName="phone"
      class="form-control"
    />
  </div>

  <div class="form-group">
    <label for="age">Age *</label>
    <input
      id="age"
      type="number"
      formControlName="age"
      class="form-control"
      [class.is-invalid]="age?.invalid && age?.touched"
    />
    <div class="invalid-feedback" *ngIf="age?.invalid && age?.touched">
      {{ getErrorMessage('age') }}
    </div>
  </div>

  <div formGroupName="address">
    <h3>Address</h3>
    
    <div class="form-group">
      <label for="street">Street *</label>
      <input
        id="street"
        type="text"
        formControlName="street"
        class="form-control"
      />
    </div>

    <div class="form-group">
      <label for="city">City *</label>
      <input
        id="city"
        type="text"
        formControlName="city"
        class="form-control"
      />
    </div>

    <div class="form-group">
      <label for="zipCode">ZIP Code *</label>
      <input
        id="zipCode"
        type="text"
        formControlName="zipCode"
        class="form-control"
      />
    </div>
  </div>

  <button
    type="submit"
    class="btn btn-primary"
    [disabled]="isSubmitting"
  >
    {{ isSubmitting ? 'Submitting...' : 'Submit' }}
  </button>
</form>
```

## Example: Custom Validator

```typescript
// validators/custom-validators.ts
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

export class CustomValidators {
  static passwordMatch(passwordField: string, confirmPasswordField: string): ValidatorFn {
    return (control: AbstractControl): ValidationErrors | null => {
      const password = control.get(passwordField);
      const confirmPassword = control.get(confirmPasswordField);

      if (!password || !confirmPassword) {
        return null;
      }

      return password.value === confirmPassword.value 
        ? null 
        : { passwordMismatch: true };
    };
  }

  static noWhitespace(): ValidatorFn {
    return (control: AbstractControl): ValidationErrors | null => {
      const isWhitespace = (control.value || '').trim().length === 0;
      return isWhitespace ? { whitespace: true } : null;
    };
  }
}
```

## Module Setup

```typescript
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { ReactiveFormsModule } from '@angular/forms';
import { HttpClientModule } from '@angular/common/http';

import { AppComponent } from './app.component';
import { UserFormComponent } from './components/user-form/user-form.component';

@NgModule({
  declarations: [
    AppComponent,
    UserFormComponent,
  ],
  imports: [
    BrowserModule,
    ReactiveFormsModule,
    HttpClientModule,
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```
