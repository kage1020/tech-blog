---
title: "ボタン"
---

| light | dark |
| --- | --- |
| ![](/images/digital-component-design/button-light.png) | ![](/images/digital-component-design/button-dark.png) |

## 実装

:::details Button.tsx

各プロパティの色は以下のように設定しました．

- Active時の色を`blue-1100`に設定しました．
- `disabled`時のコントラスト比が`3.0`未満になるため，`disabled`時の背景色，文字色を変更しました．

lightモードの色組み合わせ

| property | state | `primary` | `secondary` | `tertiary` |
| --- | --- | --- | --- | --- |
| `color` | default, Focus | `white` | `blue-900` | `blue-900` |
| `color` | hover | `white` | `blue-1000` | `blue-1000` |
| `color` | active | `white` | `blue-1100` | `blue-1100` |
| `color` | disabled | `white` | `grey-420` | `grey-420` |
| `border-color` | default, Focus | `-` | `blue-900` | `-` |
| `border-color` | hover | `-` | `blue-1000` | `-` |
| `border-color` | active | `-` | `blue-1100` | `-` |
| `border-color` | disabled | `-` | `grey-420` | `-` |
| `background-color` | default, Focus | `blue-900` | `white` | `transparent` |
| `background-color` | hover | `blue-1000` | `blue-200` | `blue-200` |
| `background-color` | active | `blue-1100` | `blue-300` | `blue-300` |
| `background-color` | disabled | `grey-420` | `white` | `transparent` |

darkモードの色組み合わせ

| property | state | `primary` | `secondary` | `tertiary` |
| --- | --- | --- | --- | --- |
| `color` | default, Focus | `white` | `blue-200` | `blue-200` |
| `color` | hover | `white` | `blue-300` | `blue-300` |
| `color` | active | `white` | `blue-400` | `blue-400` |
| `color` | disabled | `white` | `grey-536` | `grey-536` |
| `border-color` | default, Focus | `-` | `blue-200` | `-` |
| `border-color` | hover | `-` | `blue-300` | `-` |
| `border-color` | active | `-` | `blue-400` | `-` |
| `border-color` | disabled | `-` | `grey-536` | `-` |
| `background-color` | default, Focus | `blue-600` | `transparent` | `transparent` |
| `background-color` | hover | `blue-700` | `blue-300/20` | `blue-300/20` |
| `background-color` | active | `blue-800` | `blue-400/20` | `blue-400/20` |
| `background-color` | disabled | `grey-536` | `white` | `transparent` |

```tsx
import { tv, cn } from "@/libs/util"
import { ButtonProps } from "@/types"

const style = tv({
  base: "rounded-sm text-xxs-mobile focus-visible:rounded-[12px] focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-4 focus-visible:outline-yellow-700",
  variants: {
    size: {
      "x-small": "my-1 h-7 min-w-[72px] px-1 py-[7px] text-xxxs",
      small: "my-0.5 h-9 min-w-20 px-[12px] py-[6px]",
      medium: "h-12 min-w-24 px-2 py-[12px]",
      large: "h-14 min-w-[136px] p-2",
    },
    variant: {
      primary: cn([
        "bg-blue-900 text-white dark:bg-blue-600",
        "hover:bg-blue-1000 dark:hover:bg-blue-700",
        "active:bg-blue-1100 dark:active:bg-blue-800",
        "disabled:bg-grey-420 dark:disabled:bg-grey-536",
      ]),
      secondary: cn([
        "border border-blue-900 bg-white text-blue-900 dark:border-blue-200 dark:bg-transparent dark:text-blue-200",
        "hover:border-blue-1000 hover:bg-blue-200 hover:text-blue-1000 dark:hover:border-blue-300 dark:hover:bg-blue-300/20 dark:hover:text-blue-300",
        "active:border-blue-1100 active:text-blue-1100 active:bg-blue-300 dark:active:border-blue-400 dark:active:bg-blue-400/20 dark:active:text-blue-400",
        "disabled:border-grey-420 disabled:bg-white disabled:text-grey-420 dark:disabled:border-grey-536 dark:disabled:bg-transparent dark:disabled:text-grey-536",
      ]),
      tertiary: cn([
        "text-blue-900 underline dark:text-blue-200",
        "hover:bg-blue-200 hover:text-blue-1000 dark:hover:bg-blue-300/20 dark:hover:text-blue-300",
        "active:bg-blue-300 active:text-blue-1100 dark:active:bg-blue-400/20 dark:active:text-blue-400",
        "disabled:bg-transparent disabled:text-grey-420 dark:disabled:bg-transparent dark:disabled:text-grey-536",
      ]),
    },
  },
})

export default function Button({
  className,
  size = "large",
  variant = "primary",
  children,
  ...props
}: ButtonProps) {
  return (
    <button
      className={style({ size, variant, className })}
      {...props}
      data-testid="button"
    >
      {children}
    </button>
  )
}
```

:::
