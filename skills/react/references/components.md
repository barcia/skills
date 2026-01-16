# Components and Styling

## Best Practices

### 1. Colocate Related Code
Keep components, styles, state close to where used. Improves performance by reducing unnecessary re-renders.

### 2. Extract Instead of Nested Render Functions

```tsx
// Bad - hard to maintain
function Component() {
  function renderItems() {
    return <ul>...</ul>;
  }
  return <div>{renderItems()}</div>;
}

// Good - separate component
function Items() {
  return <ul>...</ul>;
}

function Component() {
  return <div><Items /></div>;
}
```

### 3. Limit Props - Use Composition

```tsx
// Bad - too many props
<Dialog title="..." content="..." onConfirm={} onCancel={} confirmText="..." />

// Good - composition via children/slots
<Dialog>
  <DialogTitle>...</DialogTitle>
  <DialogContent>...</DialogContent>
  <DialogFooter>
    <Button onClick={onCancel}>Cancel</Button>
    <Button onClick={onConfirm}>Confirm</Button>
  </DialogFooter>
</Dialog>
```

### 4. Wrap Third-Party Components

Adapt to application needs. Easier to change later.

```tsx
// components/ui/link/link.tsx
import { Link as WouterLink } from "@/lib/wouter";
import { cn } from "@/utils/cn";

type LinkProps = React.ComponentProps<typeof WouterLink>;

export const Link = ({ className, ...props }: LinkProps) => {
  return <WouterLink className={cn("text-primary hover:underline", className)} {...props} />;
};
```

## Styling: Tailwind CSS

**Use when:** Rapid prototyping, utility-first approach, design systems.

```tsx
const Button = ({ variant = "default", className, ...props }) => {
  return (
    <button
      className={cn(
        "inline-flex items-center rounded-md font-medium transition-colors",
        variant === "default" && "bg-primary text-white hover:bg-primary/90",
        variant === "outline" && "border border-input bg-background hover:bg-accent",
        variant === "ghost" && "hover:bg-accent hover:text-accent-foreground",
        className
      )}
      {...props}
    />
  );
};
```

### CVA for Variants (with Tailwind)

```tsx
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/utils/cn";

const buttonVariants = cva("inline-flex items-center rounded-md font-medium transition-colors", {
  variants: {
    variant: {
      default: "bg-primary text-white hover:bg-primary/90",
      outline: "border border-input bg-background hover:bg-accent",
      ghost: "hover:bg-accent",
    },
    size: {
      default: "h-10 px-4 py-2",
      sm: "h-9 px-3 text-sm",
      lg: "h-11 px-8",
    },
  },
  defaultVariants: {
    variant: "default",
    size: "default",
  },
});

type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> & VariantProps<typeof buttonVariants>;

export const Button = ({ className, variant, size, ...props }: ButtonProps) => {
  return <button className={cn(buttonVariants({ variant, size }), className)} {...props} />;
};
```

### Tailwind CSS 4 Setup

```css
/* index.css */
@import "tailwindcss";

@layer base {
  body {
    min-height: 100dvh;
  }
}
```

## Styling: CSS Modules

**Use when:** Component isolation needed, avoiding global class conflicts, legacy projects.

```tsx
// button.module.css
.button {
  display: inline-flex;
  align-items: center;
  border-radius: 0.375rem;
  font-weight: 500;
}

.default {
  background-color: var(--color-primary);
  color: white;
}

.outline {
  border: 1px solid var(--color-border);
  background-color: transparent;
}
```

```tsx
// button.tsx
import styles from "./button.module.css";
import { cn } from "@/utils/cn";

type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> & {
  variant?: "default" | "outline";
};

export const Button = ({ variant = "default", className, ...props }: ButtonProps) => {
  return <button className={cn(styles.button, styles[variant], className)} {...props} />;
};
```

## cn Utility

Merge class names with clsx + tailwind-merge:

```typescript
// utils/cn.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

## Component Libraries

**Headless (unstyled):** Radix UI, Headless UI, react-aria, Ark UI

**Code-based (copy/paste):** ShadCN UI, Park UI

**Fully Styled:** Chakra UI, MUI, Mantine (when rapid prototyping needed)

## Storybook (Optional)

For component development and documentation:

```tsx
// button.stories.tsx
import type { Meta, StoryObj } from "@storybook/react";
import { Button } from "./button";

const meta: Meta<typeof Button> = {
  component: Button,
};
export default meta;

type Story = StoryObj<typeof Button>;

export const Default: Story = { args: { children: "Click me" } };
export const Outline: Story = { args: { variant: "outline", children: "Outline" } };
```
