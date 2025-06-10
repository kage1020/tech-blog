---
title: "Next.jsでゲーム作る (4)"
emoji: "🐷"
type: "tech"
topics: []
published: false
---

webの沼に生きるものとして一度はゲーム制作をやりたいと思った今日この頃．
Next.jsを愛してやまないので，「Next.jsはやればできる子なんだぞ」というところを見せる意味で作ってみようと思います．

前回の記事はこちらからどうぞ

https://zenn.dev/kage1020/articles/1dcce32ac956c0

https://traffic-ltd.vercel.app/

https://github.com/kage1020/TrafficLtd

:::message
この記事と実際のコードとの大きな乖離があるため，近いうちに大きく書き直します．
:::

# Markerと客を増やす

ゲーム進行は主に`play.tsx`で実行されていくため，この中の`useEffect`でゲームロジックを書いていきます．とりあえず，30秒経過するごとに，ランダムな地点をvisibleにするか何もしないようにしました．

:::details /scene/play.tsx
```tsx
...
useEffect(() => {
  if (isRunning && seconds % 30 === 0) {
    const rand = Math.random()
    if (rand < 0.2) showTrainPoint(trainKeys[Math.floor(Math.random() * trainKeys.length)])
    else if (rand < 0.4) showAirportPoint(airportKeys[Math.floor(Math.random() * airportKeys.length)])
    else if (rand < 0.6) showPortPoint(portKeys[Math.floor(Math.random() * portKeys.length)])
    else if (rand < 0.8) showHokkaidoBusPoint(busKeys[Math.floor(Math.random() * busKeys.length)])
    }
  }, [isRunning, seconds, showTrainPoint, showAirportPoint, showPortPoint, showHokkaidoBusPoint])
  ...
```
:::