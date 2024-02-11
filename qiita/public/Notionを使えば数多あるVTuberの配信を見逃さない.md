---
title: Notionを使えば数多あるVTuberの配信を見逃さない
tags:
  - API
  - RSS
  - YouTube
  - Vtuber
  - Notion
private: false
updated_at: '2022-09-07T20:24:50+09:00'
id: cbf579a1abed6ebc77e0
organization_url_name: null
slide: false
ignorePublish: false
---
# 充実したYouTube生活を送っていますか？

YouTubeっていいですよね．自分の見たいものを好きなだけいつでも見れる．ゲーム実況，料理動画，旅動画，歌，ニュース，癒し系などすべてのジャンルが詰まっているといっても過言ではないかもしれません．
さらには，最近YouTuber, VTuberなる人たちも出てきて飽きないですよね．

そう，VTuberはまさにYouTubeにおける一大コンテンツですよね．もちろんこれを読んでいるあなたもVTuber大好きですよね()
でも，あなたは少し困っているんじゃないですか？

### 推しのVTuberが多すぎる o((>ω< ))o

特に箱推しの人は1つ追うだけでも100人以上．箱が2つ，3つと増えたら，と考えると配信を追いかけるのが精いっぱいでスケジュール管理なんてできません．そんなあなたのために，NotionをViewにしたRSSリーダーの作り方を教えます．

# 環境，仕様技術

本題です．
今回のRSSリーダーの環境は以下の通りです．特に肝となるのは上2つです．

* YouTube Feed
* Google Cloud Platform
* Python(feedparser)
* Notion

今回Feedで取得したデータをNotionで見ますが，Notionでなくてもかまいません．ただNotion APIを使ってみたかっただけなのでメールやLINEとかで配信してもいいです．

# 動画情報を取得する

YouTubeから動画情報をとってくる方法として[YouTube Data API v3](https://developers.google.com/youtube/v3/docs?hl=ja)が有名ですよね．それなりに無料枠もあって便利ではあるんですが，利用規約が厄介だったりAPIキーが面倒くさかったりと手を出しづらくもあります．

例えば，YouTube Data APIの利用規約[^terms] にはこんな一文があったりします．

[^terms]: YouTube API Services - Developer Policies
https://developers.google.com/youtube/terms/developer-policies

> (III.E.4.b)
> To be clear, an API Client must not store statistics retrieved as Non-Authorized Data for more than 30 days. For example, an API Client must not store the subscriber count for a YouTube channel for more than 30 days without authorization from the channel owner.
>
> (意訳)
> つまり，チャンネル運営じゃないやつが30日以上統計情報を保存すんなよ！例えば，(API経由の)チャンネル登録者数の情報を運営者の許可なく保存することはだめだからね！

こんな文言今初めて知りました．そういうわけで，気軽にAPIを使えないので以下のFeedを使います．

```
https://www.youtube.com/feeds/videos.xml?channel_id={CHANNEL_ID}
```

`CHANNEL_ID`を指定してあげればこんなのが返ってきます．HIKAKINさんのfeedです．

```xml
<!-- https://www.youtube.com/feeds/videos.xml?channel_id=UCZf__ehlCEBPop-_sldpBUQ -->
<feed xmlns:yt="http://www.youtube.com/xml/schemas/2015" xmlns:media="http://search.yahoo.com/mrss/" xmlns="http://www.w3.org/2005/Atom">
  <link rel="self" href="http://www.youtube.com/feeds/videos.xml?channel_id=UCZf__ehlCEBPop-_sldpBUQ"/>
  <id>yt:channel:UCZf__ehlCEBPop-_sldpBUQ</id>
  <yt:channelId>UCZf__ehlCEBPop-_sldpBUQ</yt:channelId>
  <title>HikakinTV</title>
  <link rel="alternate" href="https://www.youtube.com/channel/UCZf__ehlCEBPop-_sldpBUQ"/>
  <author>
    <name>HikakinTV</name>
    <uri>https://www.youtube.com/channel/UCZf__ehlCEBPop-_sldpBUQ</uri>
  </author>
  <published>2011-07-19T11:31:43+00:00</published>
  <entry>
    <id>yt:video:ESfa_Scr7RQ</id>
    <yt:videoId>ESfa_Scr7RQ</yt:videoId>
    <yt:channelId>UCZf__ehlCEBPop-_sldpBUQ</yt:channelId>
    <title>湖で超巨大生物を捕獲するおろちんゆーとヒカキン #shorts</title>
    <link rel="alternate" href="https://www.youtube.com/watch?v=ESfa_Scr7RQ"/>
    <author>
      <name>HikakinTV</name>
      <uri>https://www.youtube.com/channel/UCZf__ehlCEBPop-_sldpBUQ</uri>
    </author>
    <published>2022-09-06T09:03:47+00:00</published>
    <updated>2022-09-06T10:20:10+00:00</updated>
    <media:group>
      <media:title>湖で超巨大生物を捕獲するおろちんゆーとヒカキン #shorts</media:title>
      <media:content url="https://www.youtube.com/v/ESfa_Scr7RQ?version=3" type="application/x-shockwave-flash" width="640" height="390"/>
      <media:thumbnail url="https://i2.ytimg.com/vi/ESfa_Scr7RQ/hqdefault.jpg" width="480" height="360"/>
      <media:description>◆チャンネル登録はこちら↓ http://www.youtube.com/user/hikakintv?sub_confirmation=1 ◆ツイッター https://twitter.com/hikakin ◆インスタグラム https://instagram.com/hikakin/ ◆TikTok https://vt.tiktok.com/BTWxUN/ ◆ヒカキンゲームズ http://www.youtube.com/hikakingames ◆ビートボックス動画のHIKAKINチャンネル http://www.youtube.com/HIKAKIN ◆ラフな動画のHikakinBlog http://www.youtube.com/hikakinblog ◆ヒカキンLINEスタンプはこちら https://store.line.me/stickershop/product/1022677/ja ◆ヒカキンLINE公式アカウント ●友達登録はこちら↓ http://line.naver.jp/ti/p/%40hikakin #shorts #おろちんゆー #ヒカキン</media:description>
      <media:community>
        <media:starRating count="7173" average="5.00" min="1" max="5"/>
        <media:statistics views="85281"/>
      </media:community>
    </media:group>
  </entry>
  <!-- 以下entryの繰り返し -->
</feed>
```

チャンネル名や動画タイトル，サムネイル，視聴回数まで取得できます．おいしいです．

規約を探しても見つからなかったので正々堂々とは使えないですが，白よりのグレーだと信じて使います．
節度を守ってやれば大丈夫なはず．．．((((；ﾟДﾟ))))ｶﾞｸｶﾞｸﾌﾞﾙﾌﾞﾙ

しかし，ここで1つ問題があります．

#### 推しの`CHANNEL_ID`を知らないんだが？

そんなあなたに朗報です．YouTubeのチャンネルURLを渡すだけでFeedを作成してくれる神のようなサイトがあります．

https://berss.com/feed/

[Firewrench](https://www.firewrench.com/)というGoogleとも取引経験がある素晴らしい日本の企業が運営しているサイトなのでありがたく使わせていただきましょう．

(あれ，このサイト「RSSリスティング」なるものを公開している．．．便利そう．．．)

これでめでたく押しの動画情報を取得することができました．ちなみにこの情報には動画だけでなくライブ配信やプレミア公開も含まれます．

# FeedをNotionに送る

取得した情報をNotionで見れるようにします．とりあえずNotion APIの[チュートリアル](https://developers.notion.com/docs/getting-started)にしたがってtokenを発行し，データベースIDを控えときます．

後はPythonでFeedを取得し，NotionへPOSTします．Feed取得にはfeedparserを，POSTにはrequestsを使います．

```bash
pip install feedparser
```

```python
# Standard packages
import json
import requests
from datetime import datetime as dt, timezone, timedelta
import time

# Installed Packages
import feedparser

# Constants
NOTION_DATABASE_ID = 'YOUR_NOTION_DATABASE_ID'
NOTION_SECRET_TOKEN = 'YOUR_NOTION_SECRET_TOKEN'
urls = [
  # feed urls
]
JST = timezone(timedelta(hours=+9), 'JST')
headers = {
    'Authorization': 'Bearer ' + NOTION_SECRET_TOKEN,
    'Content-Type': 'application/json',
    'Notion-Version': '2022-06-28'
}


def post_page(**kwargs):
    data = {
        'parent': { 'database_id': NOTION_DATABASE_ID },
        'properties': {
            'VideoId': {
                'title': [
                    { 'text': { 'content': kwargs['video_id'] } }
                ]
            },
            'Streamer': { 
                'rich_text': [
                    {
                        'text': { 'content': kwargs['author_name'], 'link': { 'url': kwargs['author_url'] } },
                        'annotations': { 'bold': True }
                    }
                ]
            },
            'Title': {
                'rich_text': [
                    {
                        'text': { 'content': kwargs['title'], 'link': { 'url': kwargs['video_url'] } },
                        'annotations': { 'bold': True }
                    }
                ]
            },
            'Date': { 'date': { 'start': kwargs['date'] } },
            'Watched': { 'checkbox': False }
        },
        'cover': {
            'external': { 'url': f"http://img.youtube.com/vi/{kwargs['video_id']}/hqdefault.jpg" }
        }
    }
    res = requests.request(
        'POST', 
        'https://api.notion.com/v1/pages', 
        headers=headers, 
        data=json.dumps(data)
    )
    time.sleep(0.4)
    if not res.ok:
        print(res)


def is_new(video_id):
    data = {
        'filter': {
            'property': 'VideoId',
            'rich_text': { 'equals': video_id }
        }
    }
    res = requests.request(
        'POST', 
        f'https://api.notion.com/v1/databases/{NOTION_DATABASE_ID}/query',
        headers=headers,
        data=json.dumps(data)
    )
    time.sleep(0.4)
    if res.ok:
        return len(res.json()['results']) == 0
    if not res.ok:
        print(res)
        exit(1)


# execute this function in Cloud Functions
def main(event, context):
    prev = dt.now(JST) - timedelta(minutes=31)
    for url in urls:
        elements = feedparser.parse(url)
        for entry in elements.entries:
            video_id = entry['yt_videoid']
            author_name = entry['author']
            author_url = entry['authors'][0]['href']
            title = entry['title']
            video_url = entry['link']
            published = dt.fromisoformat(entry['published']).astimezone(JST)
            if prev <= published:
                if is_new(video_id):
                    post_page(video_id=video_id, author_name=author_name, author_url=author_url, title=title, video_url=video_url, date=str(published))
            else:
                break
```

`main`関数を実行することでFeedの取得からNotionデータベースへのPOSTまで行います．引数については後述するCloud Functionsでeventとcontextを渡されるので書いてありますが，無視してもらって構わないです．
注意すべき点としてはFeedから得られる日時は世界標準時(UTC)であるため，日本標準時に直す必要があることと，データの重複が無いよう`video_id`をユニークキーとして都度データベースへ調べに行く必要があることです．これは`is_new`関数の中でPOSTでクエリを投げて調べています．この時，何度もNotionにクエリを投げてサーバーに負荷をかけてはいけないので前回実行時よりも後の動画のみPOSTするようにしています．少しネストが深くてダサいですが目をつむりましょう．
あとは好みの問題ですが，`post_page`関数にたくさんの引数が並ぶのが好きではないので今回は可変長引数で取得しています．

# 定期実行

### Cloud Functions

Notionへデータを送るプログラムができたら，あとはこれを定期実行するだけです．今回はGCPのCloud FunctionsとCloud Schedulerを使ってサクッとスケジューリングしましょう．

[GCP](https://console.cloud.google.com/)にアクセスしCloud Functionsのページで関数を新たに作成しましょう．

Cloud Functionsは最近，第2世代なるものが使えるようになり実行時間が9分から10分に伸びたらしいです．HTTPならランタイムが1時間なのですが今回はその恩恵を得られません．また，第2世代でやろうとしてもうまくいかなかったので今回は第1世代でやります．

![Web キャプチャ_7-9-2022_122831_console.cloud.google.com.jpeg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2274081/0b44be3d-7c58-2bff-c4d0-4c5b897a7f4f.jpeg)


環境を第1世代，任意の関数名とリージョンを選択，トリガータイプを「Cloud Pub/Sub」にしてトピックを追加します．トピックは何でもいいです．
あとは「ランタイム、ビルド、接続、セキュリティの設定」でタイムアウトを最長の540秒に指定します．割り当てメモリはそこまで気にしなくていいと思います．

次へ進んだ後，ランタイムをPythonにしたうえで，`main.py`には上のコードを，`requirements.txt`には下のコードコピペします．
そしてデプロイししばらく待てば完了です．

```
feedparser >= 5.2.1
requests >= 2.23.0
```

### Cloud Scheduler

Cloud Schedulerのページから新しいジョブを作成し，もろもろを設定します．

![Web キャプチャ_7-9-2022_111324_console.cloud.google.com.jpeg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/2274081/246bb138-83d1-c736-d463-c6b7be8e0aec.jpeg)

頻度はお好きな間隔を設定してください．ただし，後述のNotion APIの使用制限やGCPの無料枠には注意が必要です．今回は30分おきに実行するようにしています．また，頻度を変えたらコード中の`prev`や`time.sleep(0.4)`を変えるのをお忘れなく！

その他実行内容やオプションの構成はご自由にどうぞ．

:::note warn
リージョンとトピックはCloud Functionsと同じものを選びましょう．
:::

# 運用コスト，使用制限

今回のRSSリーダーを使う上での制限，費用は以下の通りです．

* Notion
    * Notion API
        * 1秒あたり平均3回のリクエスト上限
          一時的に3回を超えた場合はその限りではなく，もし超えたとしても429が返ってきます．
* Google Cloud Platform
    * Cloud Functions
        * 毎月200万回までは無料，それ以降は100万回あたり$0.40
        * 400,000 GB 秒、200,000 GHz 秒のコンピューティング時間(今回はほぼ無視できる)
        * 1 か月あたり 5 GB のインターネット下りトラフィック(今回はほぼ無視できる)
    * Cloud Scheduler
        * 請求先アカウントあたり毎月3つのジョブ
          それ以降はジョブ1つあたり$0.10(日割りで計算されます)
          1つのスケジューラーで複数のFunctionsにトリガーできるのでそんなにたくさん使うことはないと思います．

完全無料枠でやるには，Schedulerを絶対に4個以上作らずに，1分に44回以上Functionsを実行しなければ大丈夫そうです．毎秒やると死ぬよってことです．

また，推しのVTuber何人までなら見逃さずに済むかというと，Notionの制限から今回のコードでは1動画あたり約0.8秒(0.4+0.4)かかります．30分ごとの新規動画の数が平均1個だと考えるとVTuber1人あたり0.8秒．少し多く見積もって1秒と考えると1回の実行で540人分のFeedを取得できそうです．また，1つのFunctionsは1日48回実行するので $2000000\div48\div31\simeq1344$ Functions立てられます．したがって**最大725760人**まで追いかけることができます．夢のような世界ですね！(100窓でも足りず過労死で死ぬ)

# 思考回路は同じ

頭をキャッチ―にしたいためにここまで後ろになりましたが，今回の記事はTeraさんの記事を追う形での執筆となります．ごめんなさい．こんな記事を書いといてなんですが，Teraさん後半待ってます．

https://zenn.dev/terasan/articles/d43f6311fa880f

GASを使ってやっている方もいました．

https://zenn.dev/hankei6km/articles/make-gas-library-to-setup-notion-as-feed-reader

GUI操作でFeedlyに近いこんなすばらしいRSSリーダーを作れるんだったらやるしかないですよね．
一応，VTuber界隈で有名な箱「にじさんじ」のViewを以下に公開しておきます．各自複製してMyRSSリーダーを作ってください．

https://billowy-foxtail-273.notion.site/Nijisanji-Streaming-List-0e4d192fa9ff454f91b3d111e129cbcb

それでは，良いYouTube生活を～！！

# 参考文献

https://developers.notion.com/reference/request-limits

https://cloud.google.com/functions/pricing

https://cloud.google.com/scheduler/pricing
