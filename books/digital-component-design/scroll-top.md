---
title: "スクロールトップボタン"
---

| light | dark |
| --- | --- |
| ![](/images/digital-component-design/scroll-top-light.png) | ![](/images/digital-component-design/scroll-top-dark.png) |

## 実装

:::details ScrollTop.tsx

色組み合わせ

| property | state | light | dark |
| --- | --- | --- | --- |
| `color` | default, Focus | `blue-900` | `blue-200` |
| `color` | hover | `blue-1000` | `blue-300` |
| `color` | active | `blue-1100` | `blue-400` |
| `border-color` | default, Focus | `blue-900` | `blue-200` |
| `border-color` | hover | `blue-1000` | `blue-300` |
| `border-color` | active | `blue-1100` | `blue-400` |
| `background-color` | default, Focus |`white` | `transparent` |
| `background-color` | hover | `blue-200` | `blue-300/20` |
| `background-color` | active | `blue-300` | `blue-400/20` |

```tsx
import { MdArrowUpward } from "@/components/icons"
import { cn } from "@/libs/util"
import { ScrollTopButtonProps } from "@/types"

export default function ScrollTopButton({
  className,
  ...props
}: ScrollTopButtonProps) {
  return (
    <button
      className={cn(
        "rounded-full border border-blue-900 bg-white p-2 text-blue-900 hover:border-blue-1000 hover:bg-blue-200 hover:text-blue-1000 focus:outline-2 focus:outline-offset-4 active:border-blue-1100 active:bg-blue-300 active:text-blue-1100 disabled:border-yellow-700 dark:border-blue-200 dark:bg-transparent dark:text-blue-200 dark:hover:border-blue-300 dark:hover:bg-blue-300/20 dark:active:border-blue-400 dark:active:bg-blue-400/20",
        className,
      )}
      type="button"
      {...props}
      data-testid="scroll-top-button"
    >
      <MdArrowUpward />
    </button>
  )
}
```

:::
