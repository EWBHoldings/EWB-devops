# React Coding Standards

These standards apply to all React and TypeScript frontend projects within EWB.

---

## Language and Tooling

- Use **TypeScript** — JavaScript is not permitted in new code
- Use **ESLint** with `@typescript-eslint` for static analysis
- Use **Prettier** for formatting — do not manually enforce formatting in code review
- Use **Vite** or Create React App / Next.js for project scaffolding

---

## Naming Conventions

| Element | Convention | Example |
|---|---|---|
| Component file | PascalCase | `OrderSummary.tsx` |
| Component name | PascalCase | `OrderSummary` |
| Hook file | camelCase with `use` prefix | `useOrderData.ts` |
| Hook name | camelCase with `use` prefix | `useOrderData` |
| Service / utility | camelCase | `formatCurrency.ts` |
| Type / Interface | PascalCase | `OrderSummaryProps` |
| Constant | SCREAMING_SNAKE_CASE | `MAX_ITEMS_PER_PAGE` |
| CSS module class | camelCase | `styles.orderCard` |

---

## Component Structure

- One component per file
- Keep components small and focused — a component over 150 lines is a signal to extract
- Prefer function components — never use class components in new code
- Define props interfaces above the component

```tsx
interface OrderCardProps {
  orderId: string;
  status: OrderStatus;
  onSelect: (id: string) => void;
}

export function OrderCard({ orderId, status, onSelect }: OrderCardProps) {
  return (
    <div onClick={() => onSelect(orderId)}>
      <span>{orderId}</span>
      <OrderStatusBadge status={status} />
    </div>
  );
}
```

---

## TypeScript

- Enable strict mode in `tsconfig.json` (`"strict": true`)
- Do not use `any` — use `unknown` and narrow the type explicitly
- Define explicit return types for non-trivial functions
- Use discriminated unions for complex state rather than boolean flags
- Export types and interfaces from a central `types/` directory for shared use

---

## State Management

- Use `useState` and `useReducer` for local component state
- Use React Query (`@tanstack/react-query`) or SWR for server state — do not manage loading/error/data manually
- Use Context API for lightweight global state (theme, auth)
- Use Zustand or Redux Toolkit for complex global application state
- Do not store derived data in state — compute it during render or with `useMemo`

---

## Hooks

- Follow the Rules of Hooks — only call hooks at the top level, not inside conditionals or loops
- Extract complex logic into custom hooks
- Name custom hooks with the `use` prefix
- Custom hooks should have a single clear purpose

---

## Folder Structure

```
src/
├── components/         # Reusable, presentational components
│   └── OrderCard/
│       ├── OrderCard.tsx
│       ├── OrderCard.test.tsx
│       └── index.ts
├── pages/              # Route-level components (one per page)
├── hooks/              # Custom hooks
├── services/           # API calls and data fetching logic
├── store/              # Global state management
├── types/              # Shared TypeScript types and interfaces
└── utils/              # Pure utility functions
```

Co-locate test files with the component or module they test.

---

## API Calls

- All API communication goes through `services/` — components do not call `fetch` directly
- Use React Query for data fetching; define query keys as constants
- Handle loading and error states explicitly — do not render nothing on error or loading

---

## Testing

- Use **React Testing Library** and **Jest** (or Vitest)
- Test user behaviour, not implementation details
- Do not test internal state or call internal methods
- Prefer `getByRole`, `getByLabelText`, and `getByText` over `getByTestId`
- Write integration-style tests that exercise the component as a user would

```tsx
test('displays order status correctly', () => {
  render(<OrderCard orderId="ORD-001" status="pending" onSelect={vi.fn()} />);
  expect(screen.getByText('ORD-001')).toBeInTheDocument();
  expect(screen.getByText('Pending')).toBeInTheDocument();
});
```
