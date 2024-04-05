---
title: "セレクター"
---

| light | dark |
| --- | --- |
| ![](/images/digital-component-design/selector-light.png) | ![](/images/digital-component-design/selector-dark.png) |

## 実装

:::details Selector.tsx

```tsx
import { cn } from "@/libs/util"
import { SelectorProps } from "@/types"

export default function Selector({
  className,
  label,
  options,
  supportText,
  errorText,
  required = false,
  disabled = false,
  ...props
}: SelectorProps) {
  if (options.length <= 5 && process.env.NODE_ENV === "development") {
    console.warn(
      "The number of options in the selector is less than 6. Consider using a radio button instead.",
    )
  }

  return (
    <label
      className={cn([
        "relative block space-y-1 text-grey-900 dark:text-grey-50",
        className,
      ])}
    >
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
      <div className="relative flex items-center justify-between after:pointer-events-none after:absolute after:right-4 after:z-10 after:h-2 after:w-2 after:rotate-45 after:border-b-2 after:border-r-2 after:content-['']">
        <select
          className="relative w-full appearance-none rounded-sm border border-grey-900 bg-white py-[12px] pl-2 pr-5 invalid:border-red-800 focus-visible:rounded-[12px] focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-4 focus-visible:outline-yellow-700 disabled:border-grey-420 disabled:bg-grey-50 dark:bg-grey-700 dark:invalid:border-red-500 dark:disabled:border-grey-700 dark:disabled:bg-grey-800 dark:disabled:text-grey-420"
          required={required}
          disabled={disabled}
          {...props}
        >
          {options.map((option) => (
            <option
              key={option.value}
              value={option.value}
              selected={option.selected}
              disabled={option.disabled}
            >
              {option.label}
            </option>
          ))}
        </select>
      </div>
      {errorText && <p className="text-sup-l text-red-800">{errorText}</p>}
    </label>
  )
}
```
