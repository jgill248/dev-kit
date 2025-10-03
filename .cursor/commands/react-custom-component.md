# React Custom Component

## Description
Guide for creating reusable React custom components with TypeScript, following best practices for props, styling, and composition.

## Style Rules
- Use functional components with TypeScript
- Define prop interfaces with clear types
- Use descriptive component and prop names
- Implement proper prop destructuring
- Add JSDoc comments for complex components
- Use React.FC or explicit return types
- Handle optional props with default values
- Implement proper event handlers with correct types

## Example

```typescript
import React from 'react';
import './Button.css';

interface ButtonProps {
  /** The button label text */
  label: string;
  /** Click handler */
  onClick: () => void;
  /** Button variant style */
  variant?: 'primary' | 'secondary' | 'danger';
  /** Whether the button is disabled */
  disabled?: boolean;
  /** Additional CSS classes */
  className?: string;
}

/**
 * A reusable button component with multiple variants
 */
export const Button: React.FC<ButtonProps> = ({
  label,
  onClick,
  variant = 'primary',
  disabled = false,
  className = '',
}) => {
  return (
    <button
      className={`btn btn-${variant} ${className}`}
      onClick={onClick}
      disabled={disabled}
      type="button"
    >
      {label}
    </button>
  );
};
```

## Usage

```typescript
import { Button } from './components/Button';

function App() {
  const handleClick = () => {
    console.log('Button clicked!');
  };

  return (
    <div>
      <Button label="Click Me" onClick={handleClick} />
      <Button label="Delete" onClick={handleClick} variant="danger" />
      <Button label="Disabled" onClick={handleClick} disabled />
    </div>
  );
}
```
