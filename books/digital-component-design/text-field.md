---
title: "テキスト入力"
---

| light | dark |
| --- | --- |
| ![](/images/digital-component-design/text-field-light.png) | ![](/images/digital-component-design/text-field-dark.png) |

## 実装

:::details TextField.tsx

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
import { cn } from "@/libs/util"
import { TextFieldProps } from "@/types"

export default function TextField({
  className,
  label,
  supportText,
  errorText,
  required = false,
  disabled = false,
  ...props
}: TextFieldProps) {
  return (
    <div className={cn("text-grey-900", className)}>
      <label className="block space-y-1">
        {label && (
          <p
            className={cn([
              "text-label-1 flex items-center space-x-1",
              disabled && "text-grey-420",
            ])}
          >
            <span>{label}</span>
            {required && <span className="text-sup-l text-red-800">必須</span>}
          </p>
        )}
        {supportText && (
          <p className={cn("text-sup-l", disabled && "text-grey-420")}>
            {supportText}
          </p>
        )}
        <input
          className={cn([
            "w-full rounded-sm border border-grey-900 bg-white px-2 py-[12px] text-base-l",
            "invalid:border-red-800",
            "focus-visible:rounded-[12px] focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-4 focus-visible:outline-yellow-700",
            "disabled:border-grey-420 disabled:bg-grey-50",
          ])}
          required={required}
          disabled={disabled}
          {...props}
        />
        {errorText && <p className="text-sup-l text-red-800">{errorText}</p>}
      </label>
    </div>
  )
}
```

:::
