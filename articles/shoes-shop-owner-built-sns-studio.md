---

title: "靴屋の店主が、日々のSNS投稿が面倒すぎてThreads同時投稿ツールを自作した話"

emoji: "🚀"

type: "tech" # tech: 技術記事 / idea: アイデア記事

topics: ["個人開発", "saas", "threads", "instagram", "python"]

published: true

---



![SNS Studio — 靴屋の店主が作った、Threads対応マルチSNS投稿ツール](/images/ig_post_sns_studio_v2.png)

## この記事で得られるもの（30秒で読める要点）



**この記事では、神奈川で高級靴専門店「KING of SHOES」を経営している私が、Instagram・Threads・Facebook の3SNS同時投稿ツール「SNS Studio」を、3ヶ月で自作した過程を全公開します。**



私自身、本職は靴屋でエンジニアではありません。Typefully・SocialDog・Buffer……既存のSNS管理ツールをすべて試して、**Threads非対応・日本語UIなし・運用コスト高**でどれも私の課題を解決してくれず、最終的にAI支援ツール（Claude Code）と二人三脚で自作するに至りました。



**この記事は、こんな方に向けて書きました：**



- Instagram・Threads・Facebook の投稿に毎日30〜60分以上取られている **個人事業主・副業クリエイター**

- 既存のSNS管理ツール（Typefully・SocialDog・Buffer）を試したけど **「これじゃない感」** を持っている方

- 「自分も自分のためにツールを作りたいけど、エンジニアじゃないから無理」と諦めている **非エンジニアの事業主**



**読み終えたあとに、あなたが手にしているもの：**



- 月18.5時間のSNS手作業を **月1時間に圧縮するための具体的な選択肢**

- 完全無料（月額¥0）で個人開発SaaSを運用する **技術選定の生事例**

- Meta Graph API（Instagram / Facebook / Threads）の **2026年現在の落とし穴と対処法**

- 「非エンジニアが3ヶ月で実用SaaSを作る」という **新しい選択肢の実例**



3,000文字弱の記事ですが、5分で読めます。記事末尾で **β版テスター先着10名（完全無料）を募集** していますので、SNS運用にお困りの方はぜひご検討ください。



---



## 1. 既存ツールが「全部ダメだった」理由



SaaSを作る前に、当然ながら既存ツールを探しました。結論から言うと**全部ダメだった**のです。



### Typefully：日本語UIなし・Instagram非対応



海外のスレッド型SNS投稿管理ツール。X（Twitter）スレッドの作成に特化していて、米国では月間収益$100,000を超える成功SaaSです。



**ダメだった点**：

- 日本語UIがない（英語のみ）

- Instagram投稿に対応していない

- Threads非対応

- 月額$15〜と個人事業主には少し高い



### SocialDog：X中心で他SNSが弱い



国内では人気の老舗ツール。



**ダメだった点**：

- X（Twitter）中心の設計でInstagram運用機能が薄い

- Threads非対応

- 月額¥980〜



### Buffer：Instagramカルーセル投稿が難しい



海外で有名なマルチSNSツール。



**ダメだった点**：

- 日本語サポートが弱い

- Instagramカルーセル投稿の設定が直感的でない

- Threads非対応



### 共通の致命的問題：**Threads非対応**



私が投稿で重要視していたのは Threads でした。Meta が2023年7月にローンチして、2024年6月に Graph API を公開したばかりの**新しいSNS**です。



ところが、2026年5月現在でも、**Threadsに対応している多SNS同時投稿ツールはほぼ存在しません**。Typefully は2025年に対応開始しましたが日本語UIなし。SocialDog や Buffer はまだ未対応です。



「日本語UIで Threads に対応していて、Instagram・Facebook・X もまとめて投稿できるツール」が**世の中に存在しない**ということに気づいたのが、自作を決意した瞬間でした。



---



## 2. 「自分で作るしかない」と決断した日



2026年2月、KING of SHOES の月次SNS運用工数を計算しました。



| SNS | 1日の投稿数 | 1投稿あたりの工数 | 月間合計 |

|---|---|---|---|

| Instagram | 1〜2投稿 | 5分 | 5時間 |

| X | 3〜5投稿 | 3分 | 6時間 |

| Threads | 1〜2投稿 | 5分 | 5時間 |

| Facebook | 1投稿 | 5分 | 2.5時間 |

| **合計** | **6〜10投稿** | - | **18.5時間/月** |



月18.5時間 = 約2.3営業日分。これを**自分で作ったツールで月1時間以下に圧縮できれば、年間210時間（26営業日分）の節約**になります。



「3ヶ月開発して月17時間以上節約できるなら、ROIは無限大に近い！」と考え、開発をスタートしました。



---



![完全無料で運用するSNS Studioの技術スタック構成（Render + Flask + imgBB + Meta Graph API = 月額$0）](/images/zenn_02_tech_stack.png)

## 3. 技術選定：完全無料で動かす



エンジニアではない私の最大の課題は「**運用コストをゼロにすること**」でした。月¥1,000のサーバー代でも、毎月支払うのは心理的にきつい。



調べに調べて、以下の構成にたどり着きました。



### Render Free + Flask



Webアプリの本体は Render.com の **Free プラン**（$0）で動かしています。



**特徴**：

- 15分アクセスがないとスリープに入る（次のアクセスで20〜40秒待たされる）

- 月750時間まで稼働可能

- 自動デプロイ（GitHub push → 自動更新）



スリープは個人運用なら全く問題ありません（自分が触る瞬間だけ起きていればよい）。



### Flask + Jinja2 + 素のJavaScript



フロントエンドは、私のような非エンジニアでも触れる範囲の構成にしました。



```python

# webhooks/sns_studio_api.py（簡略化）

from flask import Blueprint, jsonify, request



sns_studio_bp = Blueprint('sns_studio', __name__, url_prefix='/api/sns-studio')



@sns_studio_bp.route('/drafts/<draft_id>/post-now', methods=['POST'])

def post_now(draft_id):

    """下書きを即時投稿する（Graph APIで同期実行）。"""

    draft = get_draft(draft_id)

    # IG / FB / Threads それぞれに Graph API で投稿

    # ...

    return jsonify({'status': 'posted'})

```



React や Vue は使わず、素の JavaScript で書きました。私のような非エンジニアでも、Claude Code に「ボタン押したらこの API を叩いて」と指示すれば動くものができました。



### imgBB（画像ホスティング）



画像は imgBB の無料枠（月32,000リクエスト）にアップロードしています。Instagram Graph API は **公開URLからしか画像を読み込まない** という縛りがあるため、画像ホスティングが必須でした。



代替案として catbox.moe も試しましたが、Render の IP からは時々失敗するため、imgBB をメインに採用しました。



### Meta Graph API（Instagram / Facebook / Threads）



3つのSNSすべてに **Meta Graph API直接投稿** で対応しています。



```python

# Instagram投稿の例

def instagram_post(image_url: str, caption: str):

    # Step 1: メディアコンテナ作成

    container = api_post(f'{ig_user_id}/media', {

        'image_url': image_url,

        'caption': caption,

    })

    creation_id = container.get('id')

    

    # Step 2: 公開

    publish = api_post(f'{ig_user_id}/media_publish', {

        'creation_id': creation_id,

    })

    return publish.get('id')

```



Threads も同じパターンで実装できます（`threads_user_id/threads` でコンテナ作成 → `threads_user_id/threads_publish` で公開）。



### 月額コスト



| 項目 | コスト |

|---|---|

| Render Free | $0 |

| imgBB Free | $0 |

| Meta Graph API | $0（無料） |

| ドメイン | 取らない（onrender.com サブドメインを使用） |

| **合計** | **$0** |



**月額$0、年額$0**。これが個人開発SaaSの強みです。



---



## 4. Threads API との戦い：3つの罠



Threads API は2024年6月に公開されたばかりの新しいAPIで、ドキュメントが薄く、地雷だらけでした。



### 罠1：`threads_content_publish` 権限が別途必要



最初に作った時、`threads_basic`（プロフィール情報取得）だけのトークンで投稿しようとしたら、こんなエラーが返ってきました：



```

Application does not have permission for this action

```



「権限不足」というだけで、**何の権限が足りないかは教えてくれない**のが Meta API の地獄です。



調べたところ、投稿には `threads_content_publish` という別の権限が必要で、Meta App Dashboard で Use Case として **明示的に追加 → トークン再発行** が必要でした。



### 罠2：レスポンスの key が `media_id` で `id` ではない



Instagram も Facebook も、投稿成功時のレスポンスは `{'id': '123456'}` の形式でした。なので私は Threads も同じだと思って `result.get('id')` で取り出していたのですが、**Threads だけ `result.get('media_id')`** だったのです。



そのせいで「投稿は成功してるのにIDが保存されない → エンゲージメント取得ができない」という地味な不具合に2日ハマりました。



### 罠3：トークン再発行で User ID が変わる（場合がある）



権限追加後にトークンを再発行したら、なぜか今度はこんなエラー：



```

Object with ID '35158030977128603' does not exist, cannot be loaded due to missing permissions

```



User ID自体が古い情報のまま環境変数に残っていて、新トークンとミスマッチを起こしていました。



これを根本対策するため、**投稿前に `https://graph.threads.net/v1.0/me` を叩いて User ID を自動取得して上書きする**ロジックを入れています：



```python

# トークンから正しい User ID を自動取得（環境変数のIDが古い場合の救済）

r = requests.get('https://graph.threads.net/v1.0/me',

                 params={'fields': 'id', 'access_token': threads_access_token},

                 timeout=10)

if r.ok:

    auto_id = r.json().get('id', '')

    if auto_id and auto_id != threads_user_id:

        threads_user_id = auto_id  # 環境変数より優先

```



この「自動補正」のおかげで、今後トークンを再発行する度に環境変数を更新しなくて済むようになりました。



---



## 5. 楽観的UI（Optimistic UI）でユーザー体験を改善



開発の終盤、自分で使ってみて気になった点：**写真添付に10秒かかる**こと。



サーバー側の処理は ①画像受信 → ②imgBB アップロード → ③検証 → ④URL返却 で、合計5〜10秒かかります。その間、画面に何も表示されないのは「壊れたかな？」と感じる時間でした。



そこで採用したのが **楽観的UI（Optimistic UI）** パターンです。



```javascript

async function onImageSelected(event) {

  const file = event.target.files[0];

  if (!file) return;

  

  // 1) ローカルプレビューを即時挿入

  const localUrl = URL.createObjectURL(file);

  d.image_urls.push(localUrl);

  renderEditorScroll();  // 即座にプレビュー表示！

  

  // 2) 裏でアップロード（5〜10秒）

  const formData = new FormData();

  formData.append('image', file);

  const r = await fetch('/api/upload-image', { method: 'POST', body: formData });

  const data = await r.json();

  

  // 3) 完了したら本番URLに差し替え

  if (data.url) {

    const i = d.image_urls.indexOf(localUrl);

    d.image_urls[i] = data.url;

    URL.revokeObjectURL(localUrl);

    renderEditorScroll();

  }

}

```



この変更だけで、**体感的にゼロ秒で写真が表示されるようになりました**。Twitter や Instagram と同じUXです。



---



## 6. 開発で一番苦労したこと



技術的には Meta API の権限地獄が一番きつかったのですが、それと同じくらい辛かったのが「**自分一人で正解がわからない開発**」でした。



会社員の個人開発と違って、私には：

- レビューしてくれる先輩エンジニアがいない

- 「これでいいの？」と聞ける同僚がいない

- フィードバックをくれるユーザーがいない



そこで頼ったのが、**Claude Code（Anthropic社のAIコーディング支援ツール）** でした。



「Threads の投稿でこのエラーが出るんだけど、どうすればいい？」「このUIの実装方法、3パターン提案して、それぞれのメリットデメリット教えて」というやり取りを3ヶ月続けて、ようやく形になりました。



---



![β版テスター先着10名・完全無料・5月10日〜6月8日応募受付中](/images/zenn_03_beta_tester_banner.png)

## 7. β版テスターを募集します（先着10名・完全無料）



長文でしたが、ここまでお読みいただきありがとうございます。



**SNS Studio は今、β版を始めます**。先着10名のテスターを完全無料で募集します。



### 対象

- Instagram / Threads / Facebook を運用している個人事業主・フリーランス・副業クリエイター

- 「複数SNSの投稿が面倒」と感じている方

- フィードバックをくださる意欲のある方（口コミでも、改善要望でも）



### 提供する内容

- 8月の正式公開（月¥1,500〜）の **完全無料先行体験**

- 開発者（私）への直接フィードバック窓口

- 正式版で値上げがあっても **テスター価格を永久維持**



### 期間

- **2026年5月10日〜6月8日**（30日間）の応募受付

- **5月中**に先行体験開始予定



### 応募方法

[β版テスター応募フォーム（30秒で完了）](https://forms.gle/Q5TbbFPCBZcyRDXS8)



または、私のXアカウント [@sns_studio](https://twitter.com/sns_studio) に DM でも構いません。「Zenn記事を読みました」と一言添えてください。



---



## おわりに：「靴屋がSaaSを作る」という選択肢



非エンジニアでも、AI支援ツールがあれば、3ヶ月で実用的なSaaSが作れます。これは2026年だから可能になった話で、私は1年前ならできませんでした。



業界の外の人間が業界の常識を破る瞬間が、これからもっと増えると思います。



「自分の業務効率化のために作ったツールが、結果的に他の業界の方も助けることになる」 — そんな小さな循環を、SNS Studio で作っていきたいです。



ご質問・コメント・フィードバック、お待ちしています！



[SNS Studio LP](https://sns-studio.onrender.com/) | [β版応募フォーム](https://forms.gle/Q5TbbFPCBZcyRDXS8) | [Twitter @sns_studio](https://twitter.com/sns_studio)



---



**🙏 もしこの記事が役に立ったら、いいね・コメントで応援していただけると励みになります。**

