# React Custom Hooks

## Description
Guide for creating custom React hooks to encapsulate reusable logic, side effects, and stateful behavior.

## Style Rules
- Name hooks with "use" prefix (e.g., useLocalStorage, useFetch)
- Return values as arrays (for useState-like hooks) or objects (for multiple values)
- Document hook parameters and return values with JSDoc
- Keep hooks focused on a single responsibility
- Use TypeScript for type safety
- Handle cleanup in useEffect return functions
- Avoid breaking rules of hooks (conditionals, loops)

## Example: useLocalStorage Hook

```typescript
import { useState, useEffect } from 'react';

/**
 * Custom hook to persist state in localStorage
 * @param key - localStorage key
 * @param initialValue - default value if key doesn't exist
 * @returns [storedValue, setValue] tuple
 */
export function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T | ((val: T) => T)) => void] {
  // Get initial value from localStorage or use default
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error loading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  // Update localStorage when state changes
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(storedValue));
    } catch (error) {
      console.error(`Error saving localStorage key "${key}":`, error);
    }
  }, [key, storedValue]);

  return [storedValue, setStoredValue];
}
```

## Example: useFetch Hook

```typescript
import { useState, useEffect } from 'react';

interface FetchState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
}

/**
 * Custom hook for data fetching with loading and error states
 * @param url - API endpoint URL
 * @returns Object with data, loading, and error states
 */
export function useFetch<T>(url: string): FetchState<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const abortController = new AbortController();

    const fetchData = async () => {
      setLoading(true);
      try {
        const response = await fetch(url, { signal: abortController.signal });
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const result = await response.json();
        setData(result);
        setError(null);
      } catch (err) {
        if (err instanceof Error && err.name !== 'AbortError') {
          setError(err);
        }
      } finally {
        setLoading(false);
      }
    };

    fetchData();

    return () => {
      abortController.abort();
    };
  }, [url]);

  return { data, loading, error };
}
```

## Usage

```typescript
import { useLocalStorage } from './hooks/useLocalStorage';
import { useFetch } from './hooks/useFetch';

function App() {
  const [name, setName] = useLocalStorage('userName', '');
  const { data, loading, error } = useFetch<User[]>('https://api.example.com/users');

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <div>
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
        placeholder="Enter name"
      />
      {data && <UserList users={data} />}
    </div>
  );
}
```
