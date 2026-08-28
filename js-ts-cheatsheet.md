# 🟨 Modern JavaScript & TypeScript Reference Cheatsheet

A modern reference guide for ES6+ syntax, asynchronous programming (Promises, `async`/`await`), TypeScript types, generics, and utility types.

---

## ⚡ ES6+ Syntax & Features

```javascript
// Destructuring & Default Parameters
const user = { name: "Alice", role: "developer", settings: { theme: "dark" } };
const { name, settings: { theme } } = user;

// Array Transformation Methods (.map, .filter, .reduce)
const numbers = [1, 2, 3, 4, 5];
const evenSquared = numbers
  .filter(n => n % 2 === 0)
  .map(n => n ** 2); // [4, 16]

const sum = numbers.reduce((acc, curr) => acc + curr, 0); // 15

// Optional Chaining (?.) & Nullish Coalescing (??)
const apiPort = user.config?.server?.port ?? 3000;
```

---

## 🔄 Asynchronous JS & Promises

```javascript
// Promises & Parallel Execution (Promise.all / Promise.allSettled)
async function fetchDashboardData(userId) {
  try {
    const [profileRes, statsRes] = await Promise.all([
      fetch(`/api/users/${userId}`),
      fetch(`/api/users/${userId}/stats`)
    ]);

    if (!profileRes.ok || !statsRes.ok) throw new Error("HTTP request failed");

    const profile = await profileRes.json();
    const stats = await statsRes.json();
    return { profile, stats };
  } catch (error) {
    console.error("Failed to load dashboard:", error);
    throw error;
  }
}
```

---

## 📘 TypeScript Core Types & Generics

```typescript
// Interface & Type Alias
interface UserProfile {
  readonly id: string;
  name: string;
  email: string;
  role: "admin" | "editor" | "viewer"; // Union type
  bio?: string;                        // Optional field
}

// Generic Function
function firstElement<T>(arr: T[]): T | undefined {
  return arr.length > 0 ? arr[0] : undefined;
}

// Generic Constraints & Keyof Operator
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
```

---

## 🛠️ TypeScript Built-In Utility Types

```typescript
// 1. Partial<T> - Makes all properties optional
type UpdateUserDto = Partial<UserProfile>;

// 2. Required<T> - Makes all properties required
type FullUser = Required<UserProfile>;

// 3. Readonly<T> - Makes all properties immutable
type LockedUser = Readonly<UserProfile>;

// 4. Pick<T, K> - Constructs type by picking specified keys
type UserBasicInfo = Pick<UserProfile, "id" | "name" | "email">;

// 5. Omit<T, K> - Constructs type by omitting specified keys
type PublicUserProfile = Omit<UserProfile, "id">;

// 6. Record<K, T> - Map object with specified key & value types
type UserRolePermissions = Record<UserProfile["role"], string[]>;
```
