---
name: react-patterns
description: React 19 component patterns, hooks, state management, and composition
type: skill
agents: [frontend-architect]
knowledge: [react-patterns.json, component-patterns.json]
source: https://www.skillsdirectory.com/skills/gitwalter-react-patterns
---


# React Patterns Skill

Build modern React 19 applications using Server Components, hooks, state management, form handling, and component composition patterns.

## When to Use

- Building React 19 applications with Server Components
- Implementing custom hooks for reusable logic
- Managing complex state with Zustand or Jotai
- Handling forms with validation
- Creating reusable component patterns
- Optimizing component performance
- Implementing compound component patterns

## Prerequisites

```bash
npm install react@19 react-dom@19
npm install zustand jotai
npm install react-hook-form zod @hookform/resolvers
npm install @tanstack/react-query
```

## Process

### Step 1: React 19 Server Components and use() Hook

Use Server Components and the new use() hook for async data:

```tsx
// app/users/page.tsx (Server Component)
import { Suspense } from "react";
import { UserList } from "./components/UserList";

export default function UsersPage() {
  return (
    <div>
      <h1>Users</h1>
      <Suspense fallback={<div>Loading users...</div>}>
        <UserList />
      </Suspense>
    </div>
  );
}

// components/UserList.tsx (Server Component)
async function fetchUsers() {
  const res = await fetch("https://api.example.com/users");
  return res.json();
}

export async function UserList() {
  const users = await fetchUsers();

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// components/UserProfile.tsx (Client Component with use())
("use client");

import { use } from "react";

function fetchUser(id: string) {
  return fetch(`https://api.example.com/users/${id}`).then((r) => r.json());
}

export function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise);

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

### Step 2: Server Actions

Implement Server Actions for form submissions:

```tsx
// app/actions/user.ts
"use server";

import { revalidatePath } from "next/cache";
import { z } from "zod";

const createUserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
});

export async function createUser(formData: FormData) {
  const rawData = {
    name: formData.get("name"),
    email: formData.get("email"),
  };

  const validatedData = createUserSchema.parse(rawData);

  // Save to database
  const user = await db.user.create({ data: validatedData });

  revalidatePath("/users");
  return { success: true, user };
}

// components/CreateUserForm.tsx
("use client");

import { createUser } from "@/app/actions/user";
import { useTransition } from "react";

export function CreateUserForm() {
  const [isPending, startTransition] = useTransition();

  async function handleSubmit(formData: FormData) {
    startTransition(async () => {
      const result = await createUser(formData);
      if (result.success) {
        // Handle success
      }
    });
  }

  return (
    <form action={handleSubmit}>
      <input name="name" type="text" required />
      <input name="email" type="email" required />
      <button type="submit" disabled={isPending}>
        {isPending ? "Creating..." : "Create User"}
      </button>
    </form>
  );
}
```

### Step 3: Compound Components Pattern

Create reusable compound components:

```tsx
// components/Accordion.tsx
"use client";

import { createContext, useContext, useState, ReactNode } from "react";

interface AccordionContextType {
  openItems: Set<string>;
  toggleItem: (id: string) => void;
}

const AccordionContext = createContext<AccordionContextType | undefined>(
  undefined,
);

function useAccordion() {
  const context = useContext(AccordionContext);
  if (!context) {
    throw new Error("Accordion components must be used within Accordion");
  }
  return context;
}

export function Accordion({ children }: { children: ReactNode }) {
  const [openItems, setOpenItems] = useState<Set<string>>(new Set());

  const toggleItem = (id: string) => {
    setOpenItems((prev) => {
      const next = new Set(prev);
      if (next.has(id)) {
        next.delete(id);
      } else {
        next.add(id);
      }
      return next;
    });
  };

  return (
    <AccordionContext.Provider value={{ openItems, toggleItem }}>
      <div className="accordion">{children}</div>
    </AccordionContext.Provider>
  );
}

export function AccordionItem({
  id,
  children,
}: {
  id: string;
  children: ReactNode;
}) {
  return <div className="accordion-item">{children}</div>;
}

export function AccordionTrigger({
  id,
  children,
}: {
  id: string;
  children: ReactNode;
}) {
  const { openItems, toggleItem } = useAccordion();
  const isOpen = openItems.has(id);

  return (
    <button onClick={() => toggleItem(id)} className="accordion-trigger">
      {children}
      <span>{isOpen ? "−" : "+"}</span>
    </button>
  );
}

export function AccordionContent({
  id,
  children,
}: {
  id: string;
  children: ReactNode;
}) {
  const { openItems } = useAccordion();
  const isOpen = openItems.has(id);

  if (!isOpen) return null;

  return <div className="accordion-content">{children}</div>;
}

// Usage
<Accordion>
  <AccordionItem id="1">
    <AccordionTrigger id="1">Section 1</AccordionTrigger>
    <AccordionContent id="1">Content 1</AccordionContent>
  </AccordionItem>
</Accordion>;
```

### Step 4: Custom Hooks

Create reusable custom hooks:

```tsx
// hooks/useDebounce.ts
import { useState, useEffect } from "react";

export function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}

// hooks/useLocalStorage.ts
import { useState, useEffect } from "react";

export function useLocalStorage<T>(key: string, initialValue: T) {
  const [storedValue, setStoredValue] = useState<T>(() => {
    if (typeof window === "undefined") {
      return initialValue;
    }

    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      return initialValue;
    }
  });

  const setValue = (value: T | ((val: T) => T)) => {
    try {
      const valueToStore =
        value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);

      if (typeof window !== "undefined") {
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue] as const;
}
```

### Step 5: State Management with Zustand

Implement global state with Zustand:

```tsx
// stores/userStore.ts
import { create } from "zustand";
import { devtools, persist } from "zustand/middleware";

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserState {
  user: User | null;
  setUser: (user: User | null) => void;
  clearUser: () => void;
}

export const useUserStore = create<UserState>()(
  devtools(
    persist(
      (set) => ({
        user: null,
        setUser: (user) => set({ user }),
        clearUser: () => set({ user: null }),
      }),
      { name: "user-storage" },
    ),
  ),
);

// components/UserProfile.tsx
("use client");

import { useUserStore } from "@/stores/userStore";

export function UserProfile() {
  const { user, setUser } = useUserStore();

  if (!user) {
    return <div>Not logged in</div>;
  }

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

### Step 6: Form Handling with React Hook Form + Zod

Implement forms with validation:

```tsx
// components/UserForm.tsx
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const userSchema = z.object({
  name: z.string().min(1, "Name is required"),
  email: z.string().email("Invalid email address"),
  age: z.number().min(18, "Must be at least 18"),
});

type UserFormData = z.infer<typeof userSchema>;

export function UserForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<UserFormData>({
    resolver: zodResolver(userSchema),
  });

  const onSubmit = async (data: UserFormData) => {
    await fetch("/api/users", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(data),
    });
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <div>
        <label>Name</label>
        <input {...register("name")} />
        {errors.name && <span>{errors.name.message}</span>}
      </div>

      <div>
        <label>Email</label>
        <input type="email" {...register("email")} />
        {errors.email && <span>{errors.email.message}</span>}
      </div>

      <div>
        <label>Age</label>
        <input type="number" {...register("age", { valueAsNumber: true })} />
        {errors.age && <span>{errors.age.message}</span>}
      </div>

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? "Submitting..." : "Submit"}
      </button>
    </form>
  );
}
```

### Step 7: Component Composition

Use composition patterns for flexible components:

```tsx
// components/Card.tsx
interface CardProps {
  children: React.ReactNode;
  className?: string;
}

export function Card({ children, className }: CardProps) {
  return <div className={`card ${className}`}>{children}</div>;
}

export function CardHeader({ children }: { children: React.ReactNode }) {
  return <div className="card-header">{children}</div>;
}

export function CardBody({ children }: { children: React.ReactNode }) {
  return <div className="card-body">{children}</div>;
}

export function CardFooter({ children }: { children: React.ReactNode }) {
  return <div className="card-footer">{children}</div>;
}

// Usage
<Card>
  <CardHeader>
    <h3>Title</h3>
  </CardHeader>
  <CardBody>
    <p>Content</p>
  </CardBody>
  <CardFooter>
    <button>Action</button>
  </CardFooter>
</Card>;
```

## Output

- React 19 Server and Client Components
- Custom hooks for reusable logic
- State management setup (Zustand/Jotai)
- Form handling with validation
- Compound component patterns
- Composable component architecture

## Best Practices

- Use Server Components by default, Client Components only when needed
- Extract reusable logic into custom hooks
- Use Zustand or Jotai for global state, React state for local state
- Validate forms with Zod schemas
- Implement compound components for flexible APIs
- Use React.memo() and useMemo() for performance optimization
- Prefer composition over prop drilling
- Use TypeScript for type safety
- Implement error boundaries for error handling
- Use Suspense boundaries for loading states
- Optimize bundle size with code splitting
- Use React Query for server state management

## Related

- Knowledge: `knowledge/react-patterns.json`, `knowledge/component-patterns.json`
- Skill: `nextjs-development` for Next.js App Router patterns
- Skill: `frontend-testing` for React testing patterns
