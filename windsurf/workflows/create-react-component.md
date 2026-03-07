---
description: Create a new empty React component with related files
---

# Create Empty React Component

Create a new empty React component with related files.

1. Ask for the component name if it was not provided.
2. Create a folder in **PascalCase**.
3. Create these files:
   - `<ComponentName>.tsx`
   - `<ComponentName>.types.ts`
   - `<ComponentName>.module.css`
   - `<ComponentName>.test.tsx`
   - `index.ts`

4. Use this structure:

   ```text
   src/components/Button/
   ├── Button.tsx
   ├── Button.types.ts
   ├── Button.module.css
   ├── Button.test.tsx
   └── index.ts
   ```

## Component file

Create <ComponentName>.tsx.
• Use a function component
• Import styles from <ComponentName>.module.css
• Import props type from <ComponentName>.types.ts
• Use the props in the component
• Render a div with the component name
• Apply the CSS class with the component name

Example:

```typescript
import styles from "./Button.module.css";
import type { ButtonProps } from "./Button.types";

export function Button({ children }: ButtonProps) {
return <div className={styles.Button}>{children ?? "Button"}</div>;
}
```

## Types file

Create <ComponentName>.types.ts.
• Define <ComponentName>Props
• Include optional children

Example:

```typescript
import type { ReactNode } from "react";

export type ButtonProps = {
  children?: ReactNode;
};
```

## Styles file

Create <ComponentName>.module.css.
• Add one empty class with the component name

Example:

```css
.Button {
}
```

## Test file

Create <ComponentName>.test.tsx.
• Add a minimal test
• Check that the component renders

Example:

```typescript
import { render, screen } from "@testing-library/react";
import { Button } from "./Button";

test("renders Button", () => {
render(<Button />);
expect(screen.getByText("Button")).toBeInTheDocument();
});
```

## Export file

Create index.ts.
• Export the component
• Export the component props type

Example:

```typescript
export { Button } from "./Button";
export type { ButtonProps } from "./Button.types";
```
