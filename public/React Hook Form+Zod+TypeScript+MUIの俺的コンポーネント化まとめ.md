---
title: React Hook Form+Zod+TypeScript+MUIの俺的コンポーネント化まとめ
tags:
  - TypeScript
  - React
  - material-ui
  - react-hook-form
  - zod
private: false
updated_at: '2023-10-17T15:20:33+09:00'
id: e6a6336132c83a0c3cef
organization_url_name: null
slide: false
ignorePublish: false
---
# はじめに

この記事は，ここにたどり着くまでのストーリーとその実装方法の2本仕立てです．ストーリーに興味がない方は [**こちら**](#zodによるバリデーションとrhfによるstateの取得) へどうぞ．
実装はReact Hook FormとZodとTypeScriptとMUI(旧Material UI)をいい感じに組み合わせて，いい感じにコンポーネント分割するための現状の最善策です．他にいい方法等があったら教えてください．適宜修正します．

# 仲良くなれなかったReact Hook Form

私はすべてのstateを管理したい民なので，初めてReact Hook Form(以下RHF)と出会ったとき「こんなん流行らんやろ」と一蹴していました．(私がインターネットの海で観測していた限りではstate管理のメジャーオブメジャーはreduxで，recoilやjotai人気が流れつつある状況だったため，自然と状態管理するのが普通になってました．)

それからしばらくして，入力項目が数十個あるフォームを作らなければいけない開発にぶち当たったときまたRHFと出会いました．このときは私の中ではTypeScriptとMUIがブームで，MUIのpropsなどをカスタマイズしたオリジナルのコンポーネントを作成して何度も使いまわすということをしていました．もちろん，すべての入力を1つのページ内で管理するということはとても大変なのでstateを管理せずに実装しており，今回こそRHFを組み込めるのではと思い挑戦しました．しかし，MUIとRHFの複雑なGenericsの絡み合いに苦戦し断念せざるをえませんでした．

# 救世主Zod

それからまたしばらくして，私は救世主に出会いました．zod様です．この方はスキーマの定義とバリデーションという力を持っていらっしゃるのですが，なんといってもその御力を持ってRHFを容易く飼いならしてしまいました．

これまでの私は，フォームページでの記述負担を減らすため「バリデーションも含めて各入力コンポーネントの状態はコンポーネント内で完結すべき」という考えのもと以下のように実装していました．

```tsx:危険なフォーム実装
const FormPage = () => {
  const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
    e.preventDefault()
    const values = Object.fromEntries(new FormData(event.target as HTMLFormElement).entries())
    // valuesをいじくる
  }

  return (
    <form onSubmit={handleSubmit}>
      <Input name='name' /> {/* カスタムMUIコンポーネント */}
      <DateInput name='birthday' /> {/* カスタムMUIコンポーネント */}
      ...
    </form>
  )
}
```
これは，submitされるときはすべての入力項目が正しく入力されている前提があって初めて動くものでありすごく危険でした．しかし，こうでもしないと数十個の入力項目をいちいち管理しなければいけず面倒でした(リリースはしてないからヨシ！)．

これをRHFで管理しようとした私はコンポーネント化に苦しみました．インターネット上に転がっている情報はどれも1つのページ内でstateを管理しており，無駄な型定義をしていないため知らないエラーの連発．特にRHFの複雑な型やgenericsにボコボコにされ合計3日ほどつぶしたと思います．当時はgenericsを学び始めたばかりだったため，何をすればいいのかわからなかったのもあると思います．

```tsx:何も理解できてないgenerics
type InputProps = {
  name: string
  control: Control<T> // Tってなに？
}

const Input = ({ name, control }: InputProps) => {
  return (
    <Controller
      name={name}
      control={control} // Tがどうのこうのと怒られる
      render={...} // バリデーションもこの中で表示する
    />
  )
}

const FormPage = () => {
  const {
    control,  // どっち使えばいいの？
    register
  } = useForm()

  return (
    ...
    <Input {...register('name')} /> {/* どっち使えばいいの？ */}
    <Input control={control} name='name' />
    ...
  )
}
```

しかし，genericsの知識をつけzod様と出会った私は，後述の実装方法にたどり着くまでに至りました．

# zodによるバリデーションとRHFによるstateの取得

本題です．入力コンポーネントに関しては特段珍しいことはしてなくて，「genericsを正しく使おうね」ぐらいです．

```tsx:入力フィールド
import { FormControl, FormHelperText, FormLabel, OutlinedInput } from '@mui/material'
import { Controller } from 'react-hook-form'
import type { FieldValues, Path } from 'react-hook-form'

type InputProps<T extends FieldValues> = {
  name: Path<T>
  label: string
  error?: string
  control: Control<T>
}

const Input = <T extends FieldValues>({ name, label, error, control }: InputProps<T>) => {
  return (
    <Controller
      name={name}
      control={control}
      render={({ field }) => ( // このfieldがzodとRHFの管理下
        <FormControl>
          <FormLabel>{label}</FormLabel>
          <OutlinedInput 
            name={name} 
            value={field.value} 
            onChange={field.onChange} // ここで取得したvalueが吸い上げられてバリデーションチェックまでされる
            onBlur={field.onBlur}
          />
          {error && <FormHelperText error>{error}</FormHelperText>}
        </FormControl>
      )}
    />
  )
}
```

肝はRHFの初期化の仕方とJSXの書き方です．

```tsx:フォームページ
import { zodResolver } from '@hookform/resolvers/zod'
import { useForm } from 'react-hook-form'
import { z } from 'zod'

const schema = z.object({
  name: z.string(),
  birthday: z.date()
})

type InputState = z.infer<typeof schema>

const FormPage = () => {
  const {
    control,
    handleSubmit,
    formState: { errors },
  } = useForm<InputState>({
    resolver: zodResolver(schema), // zodをRHFに流し込むことでバリデーションができる
  })

  const onSubmit = (data: InputState) => {
    ...
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <Input
        name='name'
        control={control}
        error={errors.name?.message ?? undefined}
      />
    </form>
  )
}
```

本来，RHFで吸い上げた値は`onSubmit`関数内でバリデーションチェックをし，必要であればページ内でエラーを出す必要がありました．これは子コンポーネントから値を取得してまたは子コンポーネントに値を渡すという複雑な処理をしており褒められたものではありません．しかし，この動的なバリデーションチェックをzodにより静的にかつ裏側で行ってくれれば，コード上ではRHFを含んだコンポーネント内部で処理しているように見えます．そのうえエラーの管理はページ上でできます(私はコンポーネント内でやりましたが)．さらに，各入力項目に渡すpropsが`control`だけでいいというのも魅力です．

~~また，RHF入りのコンポーネントを作るにはJSXにgenericsを指定する必要がありますが，JSX記法とgenerics記法を混ぜるという奇天烈なコードによって解決できます．狂ってやがるぜ(誉め言葉)~~

MUIについては`OutlinedInput`に限らず`Radio` ~~や`DatePicker`~~ にも同様に使えるのでちょうどいい抽象化レベルだと思います．

**2023/01/31** 追記
genericsを渡さなくても`control`から推論してくれました．

`DatePicker`については，`DatePicker`の`onChange`イベントが`value`を直接渡してくるので`field.onChange`が使えません(`field.onChange`はeventを引数にとります．)．現状回避策を思いついていませんが，`MobileDatePicker`に統一したり，controlled componentsに切り替えたりするなどした方がよさそうです．

追記ここまで

# おわりに

以上，少々冗長にはなりましたがRHFとzodの組み合わせについてでした．間違い等あればご指摘ください．

### zodはいいぞ！
