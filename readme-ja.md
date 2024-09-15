# ![mdast][logo]

**M**ark**d**own **A**bstract **S**yntax **T**ree.

---

**mdast**は、マークダウンを[構文木][syntax-tree]で表現するための仕様です。
**[unist][]** を実装しています。
[CommonMark][]や[GitHub Flavored Markdown][gfm]など、複数の[マークダウン][]の種類を表現できます。

このドキュメントはリリースされていない可能性があります。
リリースされたドキュメントについては[releases][]を参照してください。
最新のリリースバージョンは[`5.0.0`][latest]です。

## 目次

- [はじめに](#はじめに)
  - [この仕様の位置づけ](#この仕様の位置づけ)
- [型](#型)
- [ノード（抽象）](#ノード抽象)
  - [`Literal`](#literal)
  - [`Parent`](#parent)
- [ノード](#ノード)
  - [`Blockquote`](#blockquote)
  - [`Break`](#break)
  - [`Code`](#code)
  - [`Definition`](#definition)
  - [`Emphasis`](#emphasis)
  - [`Heading`](#heading)
  - [`Html`](#html)
  - [`Image`](#image)
  - [`ImageReference`](#imagereference)
  - [`InlineCode`](#inlinecode)
  - [`Link`](#link)
  - [`LinkReference`](#linkreference)
  - [`List`](#list)
  - [`ListItem`](#listitem)
  - [`Paragraph`](#paragraph)
  - [`Root`](#root)
  - [`Strong`](#strong)
  - [`Text`](#text)
  - [`ThematicBreak`](#thematicbreak)
- [ミックスイン](#ミックスイン)
  - [`Alternative`](#alternative)
  - [`Association`](#association)
  - [`Reference`](#reference)
  - [`Resource`](#resource)
- [列挙型](#列挙型)
  - [`referenceType`](#referencetype)
- [コンテンツモデル](#コンテンツモデル)
  - [`Content`](#content)
  - [`FlowContent`](#flowcontent)
  - [`ListContent`](#listcontent)
  - [`PhrasingContent`](#phrasingcontent)
- [拡張機能](#拡張機能)
  - [GFM](#gfm)
  - [Frontmatter](#frontmatter)
  - [MDX](#mdx)
- [用語集](#用語集)
- [ユーティリティリスト](#ユーティリティリスト)
- [参考文献](#参考文献)
- [セキュリティ](#セキュリティ)
- [関連項目](#関連項目)
- [貢献](#貢献)
- [謝辞](#謝辞)
- [ライセンス](#ライセンス)

## はじめに

このドキュメントは、[マークダウン][markdown]を[抽象構文木][syntax-tree]として表現するための形式を定義しています。
mdast の開発は 2014 年 7 月に**[remark][]**で始まり、[unist][]が存在する前のことでした。
この仕様は[Web IDL][webidl]に似た文法で書かれています。

### この仕様の位置づけ

mdast は[unist][]を拡張し、構文木のための[ユーティリティのエコシステム][utilities]を活用しています。

mdast は[JavaScript][]と関連しており、JavaScript で準拠した構文木を扱うための豊富な[ユーティリティのエコシステム][list-of-utilities]があります。
しかし、mdast は JavaScript に限定されず、他のプログラミング言語でも使用できます。

mdast は[unified][]および[remark][]プロジェクトと関連しており、mdast 構文木はそれらのエコシステム全体で使用されています。

## 型

TypeScript を使用している場合、npm を使用して unist 型をインストールできます：

```sh
npm install @types/mdast
```

## ノード（抽象）

### `Literal`

```idl
interface Literal <: UnistLiteral {
  value: string
}
```

**Literal**（[**UnistLiteral**][dfn-unist-literal]）は、mdast の値を含む抽象インターフェースを表します。

その`value`フィールドは`string`です。

### `Parent`

```idl
interface Parent <: UnistParent {
  children: [MdastContent]
}
```

**Parent**（[**UnistParent**][dfn-unist-parent]）は、他のノードを含む mdast の抽象インターフェースを表します（[_子_][term-child]と呼ばれます）。

そのコンテンツは他の[**mdast コンテンツ**][dfn-mdast-content]のみに制限されます。

## ノード

### `Blockquote`

```idl
interface Blockquote <: Parent {
  type: 'blockquote'
  children: [FlowContent]
}
```

**Blockquote**（[**Parent**][dfn-parent]）は、他の場所から引用されたセクションを表します。

**Blockquote**は、[**フロー**][dfn-flow-content]コンテンツが期待される場所で使用できます。
そのコンテンツモデルも[**フロー**][dfn-flow-content]コンテンツです。

例えば、次のマークダウン：

```markdown
> Alpha bravo charlie.
```

以下のように変換されます：

```js
{
  type: 'blockquote',
  children: [{
    type: 'paragraph',
    children: [{type: 'text', value: 'Alpha bravo charlie.'}]
  }]
}
```

### `Break`

```idl
interface Break <: Node {
  type: 'break'
}
```

**Break**（[**Node**][dfn-node]）は、詩や住所などの改行を表します。

**Break**は、[**フレージング**][dfn-phrasing-content]コンテンツが期待される場所で使用できます。
コンテンツモデルはありません。

例えば、次のマークダウン：

```markdown
foo··
bar
```

以下のように変換されます：

```js
{
  type: 'paragraph',
  children: [
    {type: 'text', value: 'foo'},
    {type: 'break'},
    {type: 'text', value: 'bar'}
  ]
}
```

### `Code`

```idl
interface Code <: Literal {
  type: 'code'
  lang: string?
  meta: string?
}
```

**Code**（[**Literal**][dfn-literal]）は、ASCII アートやコンピュータコードなどの整形済みテキストのブロックを表します。

**Code**は、[**フロー**][dfn-flow-content]コンテンツが期待される場所で使用できます。
そのコンテンツは`value`フィールドで表されます。

このノードは[**フレージング**][dfn-phrasing-content]コンテンツの概念である[**InlineCode**][dfn-inline-code]と関連しています。

`lang`フィールドが存在する場合があります。
これはマークアップされているコンピュータコードの言語を表します。

`lang`フィールドが存在する場合、`meta`フィールドも存在する可能性があります。
これはノードに関連するカスタム情報を表します。

例えば、次のマークダウン：

```markdown
    foo()
```

以下のように変換されます：

```js
{
  type: 'code',
  lang: null,
  meta: null,
  value: 'foo()'
}
```

そして、次のマークダウン：

````markdown
```js highlight-line="2"
foo();
bar();
baz();
```
````

以下のように変換されます：

```js
{
  type: 'code',
  lang: 'javascript',
  meta: 'highlight-line="2"',
  value: 'foo()\nbar()\nbaz()'
}
```

### `Definition`

```idl
interface Definition <: Node {
  type: 'definition'
}

Definition includes Association
Definition includes Resource
```

**Definition**（[**Node**][dfn-node]）はリソースを表します。

**Definition**は、[**コンテンツ**][dfn-content]が期待される場所で使用できます。
コンテンツモデルはありません。

**Definition**には[**Association**][dfn-mxn-association]と[**Resource**][dfn-mxn-resource]のミックスインが含まれます。

**Definition**は[**LinkReferences**][dfn-link-reference]および[**ImageReferences**][dfn-image-reference]と関連付けられるべきです。

例えば、次のマークダウン：

```markdown
[Alpha]: https://example.com
```

以下のように変換されます：

```js
{
  type: 'definition',
  identifier: 'alpha',
  label: 'Alpha',
  url: 'https://example.com',
  title: null
}
```

### `Emphasis`

```idl
interface Emphasis <: Parent {
  type: 'emphasis'
  children: [PhrasingContent]
}
```

**Emphasis**（[**Parent**][dfn-parent]）は、その内容の強調を表します。

**Emphasis**は、[**フレージング**][dfn-phrasing-content]コンテンツが期待される場所で使用できます。
そのコンテンツモデルは[**フレージング**][dfn-phrasing-content]コンテンツです。

例えば、次のマークダウン：

```markdown
_alpha_ _bravo_
```

以下のように変換されます：

```js
{
  type: 'paragraph',
  children: [
    {
      type: 'emphasis',
      children: [{type: 'text', value: 'alpha'}]
    },
    {type: 'text', value: ' '},
    {
      type: 'emphasis',
      children: [{type: 'text', value: 'bravo'}]
    }
  ]
}
```

### `Heading`

```idl
interface Heading <: Parent {
  type: 'heading'
  depth: 1 <= number <= 6
  children: [PhrasingContent]
}
```

**Heading**（[**Parent**][dfn-parent]）は、セクションの見出しを表します。

**Heading**は、[**フロー**][dfn-flow-content]コンテンツが期待される場所で使用できます。
そのコンテンツモデルは[**フレージング**][dfn-phrasing-content]コンテンツです。

`depth`フィールドが存在する必要があります。
`1`の値が最も高いランクを表し、`6`が最も低いランクを表します。

例えば、次のマークダウン：

```markdown
# Alpha
```

以下のように変換されます：

```js
{
  type: 'heading',
  depth: 1,
  children: [{type: 'text', value: 'Alpha'}]
}
```

### `Html`

```idl
interface Html <: Literal {
  type: 'html'
}
```

**Html**（[**Literal**][dfn-literal]）は、生の[HTML][]フラグメントを表します。

**Html**は、[**フロー**][dfn-flow-content]または[**フレージング**][dfn-phrasing-content]コンテンツが期待される場所で使用できます。
そのコンテンツは`value`フィールドで表されます。

**Html**ノードには、有効または完全な HTML（[\[HTML\]][html]）構造である制限はありません。

例えば、次のマークダウン：

```markdown
<div>
```

以下のように変換されます：

```js
{type: 'html', value: '<div>'}
```

### `Image`

```idl
interface Image <: Node {
  type: 'image'
}

Image includes Resource
Image includes Alternative
```

**Image**（[**Node**][dfn-node]）は画像を表します。

**Image**は、[**フレージング**][dfn-phrasing-content]コンテンツが期待される場所で使用できます。
コンテンツモデルはありませんが、`alt`フィールドで説明されます。

**Image**には[**Resource**][dfn-mxn-resource]と[**Alternative**][dfn-mxn-alternative]のミックスインが含まれます。

例えば、次のマークダウン：

```markdown
![alpha](https://example.com/favicon.ico "bravo")
```

以下のように変換されます：

```js
{
  type: 'image',
  url: 'https://example.com/favicon.ico',
  title: 'bravo',
  alt: 'alpha'
}
```

### `ImageReference`

```idl
interface ImageReference <: Node {
  type: 'imageReference'
}

ImageReference includes Reference
ImageReference includes Alternative
```

**ImageReference**（[**Node**][dfn-node]）は、関連付けを通じて画像を表します。関連付けがない場合は元のソースを表します。

**ImageReference**は、[**フレージング**][dfn-phrasing-content]コンテンツが期待される場所で使用できます。
コンテンツモデルはありませんが、`alt`フィールドで説明されます。

**ImageReference**には[**Reference**][dfn-mxn-reference]と[**Alternative**][dfn-mxn-alternative]のミックスインが含まれます。

**ImageReference**は[**Definition**][dfn-definition]と関連付けられるべきです。

例えば、次のマークダウン：

```markdown
![alpha][bravo]
```

以下のように変換されます：

```js
{
  type: 'imageReference',
  identifier: 'bravo',
  label: 'bravo',
  referenceType: 'full',
  alt: 'alpha'
}
```

### `InlineCode`

```idl
interface InlineCode <: Literal {
  type: 'inlineCode'
}
```

**InlineCode**（[**Literal**][dfn-literal]）は、ファイル名、コンピュータプログラム、またはコンピュータが解析できるものなど、コンピュータコードのフラグメントを表します。

**InlineCode**は、[**フレージング**][dfn-phrasing-content]コンテンツが期待される場所で使用できます。
そのコンテンツは`value`フィールドで表されます。

このノードは[**フロー**][dfn-flow-content]コンテンツの概念である[**Code**][dfn-code]と関連しています。

例えば、次のマークダウン：

```markdown
`foo()`
```

以下のように変換されます：

```js
{type: 'inlineCode', value: 'foo()'}
```

### `Link`

```idl
interface Link <: Parent {
  type: 'link'
  children: [PhrasingContent]
}

Link includes Resource
```

**Link**（[**Parent**][dfn-parent]）はハイパーリンクを表します。

**Link**は、[**フレージング**][dfn-phrasing-content]コンテンツが期待される場所で使用できます。
そのコンテンツモデルも[**フレージング**][dfn-phrasing-content]コンテンツです。

**Link**には[**Resource**][dfn-mxn-resource]ミックスインが含まれます。

例えば、次のマークダウン：

```markdown
[alpha](https://example.com "bravo")
```

以下のように変換されます：

```js
{
  type: 'link',
  url: 'https://example.com',
  title: 'bravo',
  children: [{type: 'text', value: 'alpha'}]
}
```

### `LinkReference`

```idl
interface LinkReference <: Parent {
  type: 'linkReference'
  children: [PhrasingContent]
}

LinkReference includes Reference
```

**LinkReference**（[**Parent**][dfn-parent]）は、関連付けを通じてハイパーリンクを表します。関連付けがない場合は元のソースを表します。

**LinkReference**は、[**フレージング**][dfn-phrasing-content]コンテンツが期待される場所で使用できます。
そのコンテンツモデルも[**フレージング**][dfn-phrasing-content]コンテンツです。

**LinkReference**には[**Reference**][dfn-mxn-reference]ミックスインが含まれます。

**LinkReferences**は[**Definition**][dfn-definition]と関連付けられるべきです。

例えば、次のマークダウン：

```markdown
[alpha][Bravo]
```

以下のように変換されます：

```js
{
  type: 'linkReference',
  identifier: 'bravo',
  label: 'Bravo',
  referenceType: 'full',
  children: [{type: 'text', value: 'alpha'}]
}
```

### `List`

```idl
interface List <: Parent {
  type: 'list'
  ordered: boolean?
  start: number?
  spread: boolean?
  children: [ListContent]
}
```

**List**（[**Parent**][dfn-parent]）は項目のリストを表します。

**List**は、[**フロー**][dfn-flow-content]コンテンツが期待される場所で使用できます。
そのコンテンツモデルは[**リスト**][dfn-list-content]コンテンツです。

`ordered`フィールドが存在する場合があります。
これは項目が意図的に順序付けられている（`true`の場合）、または項目の順序が重要でない（`false`または存在しない場合）ことを表します。

`start`フィールドが存在する場合があります。
これは、`ordered`フィールドが`true`の場合、リストの開始番号を表します。

`spread`フィールドが存在する場合があります。
これは、その子の 1 つ以上が[兄弟][term-sibling]から空行で区切られていること（`true`の場合）、またはそうでないこと（`false`または存在しない場合）を表します。

例えば、次のマークダウン：

```markdown
1. foo
```

以下のように変換されます：

```js
{
  type: 'list',
  ordered: true,
  start: 1,
  spread: false,
  children: [{
    type: 'listItem',
    spread: false,
    children: [{
      type: 'paragraph',
      children: [{type: 'text', value: 'foo'}]
    }]
  }]
}
```

### `ListItem`

```idl
interface ListItem <: Parent {
  type: 'listItem'
  spread: boolean?
  children: [FlowContent]
}
```

**ListItem**（[**Parent**][dfn-parent]）は[**List**][dfn-list]の項目を表します。

**ListItem**は、[**リスト**][dfn-list-content]コンテンツが期待される場所で使用できます。
そのコンテンツモデルは[**フロー**][dfn-flow-content]コンテンツです。

`spread`フィールドが存在する場合があります。
これは、項目に空行で区切られた 2 つ以上の[_子_][term-child]が含まれていること（`true`の場合）、またはそうでないこと（`false`または存在しない場合）を表します。

例えば、次のマークダウン：

```markdown
- bar
```

以下のように変換されます：

```js
{
  type: 'listItem',
  spread: false,
  children: [{
    type: 'paragraph',
    children: [{type: 'text', value: 'bar'}]
  }]
}
```

### `Paragraph`

```idl
interface Paragraph <: Parent {
  type: 'paragraph'
  children: [PhrasingContent]
}
```

**Paragraph**（[**Parent**][dfn-parent]）は、特定のポイントやアイデアを扱う談話の単位を表します。

**Paragraph**は、[**コンテンツ**][dfn-content]が期待される場所で使用できます。
そのコンテンツモデルは[**フレージング**][dfn-phrasing-content]コンテンツです。

例えば、次のマークダウン：

```markdown
Alpha bravo charlie.
```

以下のように変換されます：

```js
{
  type: 'paragraph',
  children: [{type: 'text', value: 'Alpha bravo charlie.'}]
}
```

### `Root`

```idl
interface Root <: Parent {
  type: 'root'
}
```

**Root**（[**Parent**][dfn-parent]）はドキュメントを表します。

**Root**は[_木_][term-tree]の[_根_][term-root]として使用でき、[_子_][term-child]としては決して使用できません。
そのコンテンツモデルは[**フロー**][dfn-flow-content]コンテンツに限定されず、代わりに任意の[**mdast コンテンツ**][dfn-mdast-content]を含むことができますが、すべてのコンテンツが同じカテゴリーである必要があるという制限があります。

### `Strong`

```idl
interface Strong <: Parent {
  type: 'strong'
  children: [PhrasingContent]
}
```

**Strong**（[**Parent**][dfn-parent]）は、その内容に対する強い重要性、深刻さ、または緊急性を表します。

**Strong**は、[**フレージング**][dfn-phrasing-content]コンテンツが期待される場所で使用できます。
そのコンテンツモデルは[**フレージング**][dfn-phrasing-content]コンテンツです。

例えば、次のマークダウン：

```markdown
**alpha** **bravo**
```

以下のように変換されます：

```js
{
  type: 'paragraph',
  children: [
    {
      type: 'strong',
      children: [{type: 'text', value: 'alpha'}]
    },
    {type: 'text', value: ' '},
    {
      type: 'strong',
      children: [{type: 'text', value: 'bravo'}]
    }
  ]
}
```

### `Text`

```idl
interface Text <: Literal {
  type: 'text'
}
```

**Text**（[**Literal**][dfn-literal]）は、単なるテキストであるすべてを表します。

**Text**は、[**フレージング**][dfn-phrasing-content]コンテンツが期待される場所で使用できます。
そのコンテンツは`value`フィールドで表されます。

例えば、次のマークダウン：

```markdown
Alpha bravo charlie.
```

以下のように変換されます：

```js
{type: 'text', value: 'Alpha bravo charlie.'}
```

### `ThematicBreak`

```idl
interface ThematicBreak <: Node {
  type: 'thematicBreak'
}
```

**ThematicBreak**（[**Node**][dfn-node]）は、物語の場面転換、別のトピックへの移行、または新しいドキュメントなどの主題の区切りを表します。

**ThematicBreak**は、[**フロー**][dfn-flow-content]コンテンツが期待される場所で使用できます。
コンテンツモデルはありません。

例えば、次のマークダウン：

```markdown
---
```

以下のように変換されます：

```js
{
  type: "thematicBreak";
}
```

## ミックスイン

### `Alternative`

```idl
interface mixin Alternative {
  alt: string?
}
```

**Alternative**はフォールバックを持つノードを表します。

`alt`フィールドが存在する必要があります。
これは、ノードを意図したとおりに表現できない環境のための同等のコンテンツを表します。

### `Association`

```idl
interface mixin Association {
  identifier: string
  label: string?
}
```

**Association**は、あるノードから別のノードへの内部関係を表します。

`identifier`フィールドが存在する必要があります。
これは別のノードと一致する可能性があります。
`identifier`はソース値です：文字エスケープと文字参照は*解析されません*。
その値は正規化される必要があります。

`label`フィールドが存在する場合があります。
`label`は文字列値です：リンクの`title`やコードの`lang`と同様に機能します：文字エスケープと文字参照が解析されます。

値を正規化するには、マークダウンの空白（`[\t\n\r ]+`）をスペースに圧縮し、オプションの先頭および/または末尾のスペースを削除し、大文字小文字の折りたたみを実行します。

`identifier`の値（または`identifier`がない場合は正規化された`label`）が一意の識別子であるかどうかは、**Association**を含むノードの種類によって異なります。
例えば、[**Definition**][dfn-definition]では一意である必要がありますが、複数の[**LinkReference**][dfn-link-reference]は 1 つの定義と関連付けられるために一意でない可能性があります。

### `Reference`

```idl
interface mixin Reference {
  referenceType: string
}

Reference includes Association
```

**Reference**は、別のノードと[**関連付けられた**][dfn-mxn-association]マーカーを表します。

`referenceType`フィールドが存在する必要があります。
その値は[**referenceType**][dfn-enum-reference-type]である必要があります。
これは参照の明示性を表します。

### `Resource`

```idl
interface mixin Resource {
  url: string
  title: string?
}
```

**Resource**はリソースへの参照を表します。

`url`フィールドが存在する必要があります。
これは参照されるリソースへの URL を表します。

`title`フィールドが存在する場合があります。
これはリソースに関する助言情報を表し、ツールチップに適切なものです。

## 列挙型

### `referenceType`

```idl
enum referenceType {
  'shortcut' | 'collapsed' | 'full'
}
```

**referenceType**は参照の明示性を表します。

- **shortcut**：参照は暗黙的で、その識別子はそのコンテンツから推論されます
- **collapsed**：参照は明示的で、その識別子はそのコンテンツから推論されます
- **full**：参照は明示的で、その識別子は明示的に設定されています

## コンテンツモデル

```idl
type MdastContent = FlowContent | ListContent | PhrasingContent
```

mdast の各ノードは、類似の特性を持つノードをグループ化する 1 つ以上の**Content**カテゴリーに分類されます。

### `Content`

```idl
type Content = Definition | Paragraph
```

**Content**は、定義と段落を形成するテキストの実行を表します。

### `FlowContent`

```idl
type FlowContent =
  Blockquote | Code | Heading | Html | List | ThematicBreak | Content
```

**Flow**コンテンツはドキュメントのセクションを表します。

### `ListContent`

```idl
type ListContent = ListItem
```

**List**コンテンツはリスト内の項目を表します。

### `PhrasingContent`

```idl
type PhrasingContent = Break | Emphasis | Html | Image | ImageReference
  | InlineCode | Link | LinkReference | Strong | Text
```

**Phrasing**コンテンツはドキュメント内のテキストとそのマークアップを表します。

## 拡張機能

マークダウン構文は頻繁に拡張されます。
この仕様のゴールは、可能なすべての拡張をリストすることではありません。
ただし、頻繁に使用される拡張機能の短いリストを以下に示します。

### GFM

以下のインターフェースは[GitHub Flavored Markdown][gfm]で見られます。

#### `Delete`

```idl
interface Delete <: Parent {
  type: 'delete'
  children: [PhrasingContent]
}
```

**Delete**（[**Parent**][dfn-parent]）は、もはや正確でない、または関連性のない内容を表します。

**Delete**は、[**フレージング**][dfn-phrasing-content]コンテンツが期待される場所で使用できます。
そのコンテンツモデルは[**フレージング**][dfn-phrasing-content]コンテンツです。

例えば、次のマークダウン：

```markdown
~~alpha~~
```

以下のように変換されます：

```js
{
  type: 'delete',
  children: [{type: 'text', value: 'alpha'}]
}
```

#### `ListItem` (GFM)

```idl
interface ListItemGfm <: ListItem {
  checked: boolean?
}
```

GFM では、`checked`フィールドが存在する場合があります。
これは項目が完了している（`true`の場合）、完了していない（`false`の場合）、または不確定または適用されない（`null`または存在しない場合）ことを表します。

#### `FootnoteDefinition`

```idl
interface FootnoteDefinition <: Parent {
  type: 'footnoteDefinition'
  children: [FlowContent]
}

FootnoteDefinition includes Association
```

**FootnoteDefinition**（[**Parent**][dfn-parent]）は、ドキュメントのフロー外にある、ドキュメントに関連するコンテンツを表します。

**FootnoteDefinition**は、[**フロー**][dfn-flow-content]コンテンツが期待される場所で使用できます。
そのコンテンツモデルも[**フロー**][dfn-flow-content]コンテンツです。

**FootnoteDefinition**には[**Association**][dfn-mxn-association]ミックスインが含まれます。

**FootnoteDefinition**は[**FootnoteReferences**][dfn-footnote-reference]と関連付けられるべきです。

例えば、次のマークダウン：

```markdown
[^alpha]: bravo and charlie.
```

以下のように変換されます：

```js
{
  type: 'footnoteDefinition',
  identifier: 'alpha',
  label: 'alpha',
  children: [{
    type: 'paragraph',
    children: [{type: 'text', value: 'bravo and charlie.'}]
  }]
}
```

#### `FootnoteReference`

```idl
interface FootnoteReference <: Node {
  type: 'footnoteReference'
}

FootnoteReference includes Association
```

**FootnoteReference**（[**Node**][dfn-node]）は、関連付けを通じたマーカーを表します。

**FootnoteReference**は、[**フレージング**][dfn-phrasing-content]コンテンツが期待される場所で使用できます。
コンテンツモデルはありません。

**FootnoteReference**には[**Association**][dfn-mxn-association]ミックスインが含まれます。

**FootnoteReference**は[**FootnoteDefinition**][dfn-footnote-definition]と関連付けられるべきです。

例えば、次のマークダウン：

```markdown
[^alpha]
```

以下のように変換されます：

```js
{
  type: 'footnoteReference',
  identifier: 'alpha',
  label: 'alpha'
}
```

#### `Table`

```idl
interface Table <: Parent {
  type: 'table'
  align: [alignType]?
  children: [TableContent]
}
```

**Table**（[**Parent**][dfn-parent]）は 2 次元データを表します。

**Table**は、[**フロー**][dfn-flow-content]コンテンツが期待される場所で使用できます。
そのコンテンツモデルは[**テーブル**][dfn-table-content]コンテンツです。

ノードの[_ヘッド_][term-head]は列のラベルを表します。

`align`フィールドが存在する場合があります。
存在する場合、[**alignType**][dfn-enum-align-type]のリストである必要があります。
これは列内のセルがどのように整列されるかを表します。

例えば、次のマークダウン：

```markdown
| foo | bar |
| :-- | :-: |
| baz | qux |
```

以下のように変換されます：

```js
{
  type: 'table',
  align: ['left', 'center'],
  children: [
    {
      type: 'tableRow',
      children: [
        {
          type: 'tableCell',
          children: [{type: 'text', value: 'foo'}]
        },
        {
          type: 'tableCell',
          children: [{type: 'text', value: 'bar'}]
        }
      ]
    },
    {
      type: 'tableRow',
      children: [
        {
          type: 'tableCell',
          children: [{type: 'text', value: 'baz'}]
        },
        {
          type: 'tableCell',
          children: [{type: 'text', value: 'qux'}]
        }
      ]
    }
  ]
}
```

#### `TableCell`

```idl
interface TableCell <: Parent {
  type: 'tableCell'
  children: [PhrasingContent]
}
```

**TableCell**（[**Parent**][dfn-parent]）は、その親が[_ヘッド_][term-head]である場合は[**Table**][dfn-table]のヘッダーセルを、そうでない場合はデータセルを表します。

**TableCell**は、[**行**][dfn-row-content]コンテンツが期待される場所で使用できます。
そのコンテンツモデルは[**Break**][dfn-break]ノードを除く[**フレージング**][dfn-phrasing-content]コンテンツです。

例については、[**Table**][dfn-table]を参照してください。

#### `TableRow`

```idl
interface TableRow <: Parent {
  type: 'tableRow'
  children: [RowContent]
}
```

**TableRow**（[**Parent**][dfn-parent]）はテーブル内のセルの行を表します。

**TableRow**は、[**テーブル**][dfn-table-content]コンテンツが期待される場所で使用できます。
そのコンテンツモデルは[**行**][dfn-row-content]コンテンツです。

ノードが[_ヘッド_][term-head]である場合、それは親[**Table**][dfn-table]の列のラベルを表します。

例については、[**Table**][dfn-table]を参照してください。

#### `alignType`

```idl
enum alignType {
  'left' | 'right' | 'center' | null
}
```

**alignType**はフレージングコンテンツがどのように整列されるかを表します（[\[CSSTEXT\]][css-text]）。

- **`'left'`**: `text-align` CSS プロパティの[`left`][css-left]値を参照
- **`'right'`**: `text-align` CSS プロパティの[`right`][css-right]値を参照
- **`'center'`**: `text-align` CSS プロパティの[`center`][css-center]値を参照
- **`null`**: フレージングコンテンツはホスト環境で定義されたとおりに整列される

#### `FlowContent` (GFM)

```idl
type FlowContentGfm = FootnoteDefinition | Table | FlowContent
```

#### `ListContent` (GFM)

```idl
type ListContentGfm = ListItemGfm
```

#### `PhrasingContent` (GFM)

```idl
type PhrasingContentGfm = FootnoteReference | Delete | PhrasingContent
```

#### `RowContent`

```idl
type RowContent = TableCell
```

**Row**コンテンツは行内のセルを表します。

#### `TableContent`

```idl
type TableContent = TableRow
```

**Table**コンテンツはテーブル内の行を表します。

### Frontmatter

以下のインターフェースは YAML で見られます。

#### `Yaml`

```idl
interface Yaml <: Literal {
  type: 'yaml'
}
```

**Yaml**（[**Literal**][dfn-literal]）は、YAML（[\[YAML\]][yaml]）データシリアライゼーション言語でのドキュメントのメタデータのコレクションを表します。

**Yaml**は、[**frontmatter**][dfn-frontmatter-content]コンテンツが期待される場所で使用できます。
そのコンテンツは`value`フィールドで表されます。

例えば、次のマークダウン：

```markdown
---
foo: bar
---
```

以下のように変換されます：

```js
{type: 'yaml', value: 'foo: bar'}
```

#### `FrontmatterContent`

```idl
type FrontmatterContent = Yaml
```

**Frontmatter**コンテンツはドキュメントに関する帯域外情報を表します。

frontmatter が存在する場合、[_木_][term-tree]内の 1 つのノードに限定され、[_ヘッド_][term-head]としてのみ存在できます。

#### `FlowContent` (frontmatter)

```idl
type FlowContentFrontmatter = FrontmatterContent | FlowContent
```

### MDX

[`remark-mdx`](https://mdxjs.com/packages/remark-mdx/#syntax-tree)を参照してください。

## 用語集

[unist 用語集][glossary]を参照してください。

## ユーティリティリスト

より多くのユーティリティについては、[unist ユーティリティリスト][utilities]を参照してください。

<!--lint disable list-item-spacing-->

- [`mdast-add-list-metadata`](https://gitlab.com/staltz/mdast-add-list-metadata)
  — `list`と`listItem`ノードのメタデータを強化します
- [`mdast-util-assert`](https://github.com/syntax-tree/mdast-util-assert)
  — ノードをアサートします
- [`mdast-builder`](https://github.com/mike-north/mdast-builder)
  — 組み合わせ可能な関数で mdast 構造を構築します
- [`mdast-comment-marker`](https://github.com/syntax-tree/mdast-comment-marker)
  — コメントマーカーを解析します
- [`mdast-util-compact`](https://github.com/syntax-tree/mdast-util-compact)
  — 木をコンパクトにします
- [`mdast-util-definitions`](https://github.com/syntax-tree/mdast-util-definitions)
  — 定義ノードを見つけます
- [`mdast-util-directive`](https://github.com/syntax-tree/mdast-util-directive)
  — ディレクティブを解析および直列化します
- [`mdast-util-find-and-replace`](https://github.com/syntax-tree/mdast-util-find-and-replace)
  — テキストを検索して置換します
- [`mdast-flatten-image-paragraphs`](https://gitlab.com/staltz/mdast-flatten-image-paragraphs)
  — `paragraph`と`image`を 1 つの`image`ノードに平坦化します
- [`mdast-flatten-listitem-paragraphs`](https://gitlab.com/staltz/mdast-flatten-listitem-paragraphs)
  — `listItem`と（ネストされた）段落を 1 つの listItem ノードに平坦化します
- [`mdast-flatten-nested-lists`](https://gitlab.com/staltz/mdast-flatten-nested-lists)
  — リスト内のリストを避けるように木を変換します
- [`mdast-util-from-adf`](https://github.com/bitcrowd/mdast-util-from-adf)
  — Atlassian Document Format（ADF）から mdast 構文木を構築します
- [`mdast-util-from-markdown`](https://github.com/syntax-tree/mdast-util-from-markdown)
  — マークダウンを解析します
- [`mdast-util-frontmatter`](https://github.com/syntax-tree/mdast-util-frontmatter)
  — フロントマターを解析および直列化します
- [`mdast-util-gfm`](https://github.com/syntax-tree/mdast-util-gfm)
  — GFM を解析および直列化します
- [`mdast-util-gfm-autolink-literal`](https://github.com/syntax-tree/mdast-util-gfm-autolink-literal)
  — GFM 自動リンクリテラルを解析および直列化します
- [`mdast-util-gfm-footnote`](https://github.com/syntax-tree/mdast-util-gfm-footnote)
  — GFM 脚注を解析および直列化します
- [`mdast-util-gfm-strikethrough`](https://github.com/syntax-tree/mdast-util-gfm-strikethrough)
  — GFM 取り消し線を解析および直列化します
- [`mdast-util-gfm-table`](https://github.com/syntax-tree/mdast-util-gfm-table)
  — GFM テーブルを解析および直列化します
- [`mdast-util-gfm-task-list-item`](https://github.com/syntax-tree/mdast-util-gfm-task-list-item)
  — GFM タスクリスト項目を解析および直列化します
- [`mdast-util-gridtables`](https://github.com/syntax-tree/mdast-util-gridtables)
  — グリッドテーブルを解析および直列化します
- [`mdast-util-heading-range`](https://github.com/syntax-tree/mdast-util-heading-range)
  — マークダウン見出しを範囲として扱います
- [`mdast-util-heading-style`](https://github.com/syntax-tree/mdast-util-heading-style)
  — 見出しノードのスタイルを取得します
- [`mdast-util-hidden`](github.com/Xunnamius/unified-utils/tree/main/packages/mdast-util-hidden)
  — トランスフォーマーによってノードが見えないようにします
- [`mdast-util-math`](https://github.com/syntax-tree/mdast-util-math)
  — 数式を解析および直列化します
- [`mdast-util-mdx`](https://github.com/syntax-tree/mdast-util-mdx)
  — MDX を解析および直列化します
- [`mdast-util-mdx-expression`](https://github.com/syntax-tree/mdast-util-mdx-expression)
  — MDX 式を解析および直列化します
- [`mdast-util-mdx-jsx`](https://github.com/syntax-tree/mdast-util-mdx-jsx)
  — MDX JSX を解析および直列化します
- [`mdast-util-mdxjs-esm`](https://github.com/syntax-tree/mdast-util-mdxjs-esm)
  — MDX ESM を解析および直列化します
- [`mdast-move-images-to-root`](https://gitlab.com/staltz/mdast-move-images-to-root)
  — 画像ノードをツリーの上に移動し、ルートの直接の子にします
- [`mdast-normalize-headings`](https://github.com/syntax-tree/mdast-normalize-headings)
  — ドキュメント内に最大 1 つのトップレベル見出しがあることを保証します
- [`mdast-util-phrasing`](https://github.com/syntax-tree/mdast-util-phrasing)
  — ノードがフレージングコンテンツであるかどうかをチェックします
- [`mdast-squeeze-paragraphs`](https://github.com/syntax-tree/mdast-squeeze-paragraphs)
  — 空の段落を削除します
- [`mdast-util-toc`](https://github.com/syntax-tree/mdast-util-toc)
  — ツリーから目次を生成します
- [`mdast-util-to-hast`](https://github.com/syntax-tree/mdast-util-to-hast)
  — hast に変換します
- [`mdast-util-to-markdown`](https://github.com/syntax-tree/mdast-util-to-markdown)
  — マークダウンを直列化します
- [`mdast-util-to-nlcst`](https://github.com/syntax-tree/mdast-util-to-nlcst)
  — nlcst に変換します
- [`mdast-util-to-string`](https://github.com/syntax-tree/mdast-util-to-string)
  — ノードのプレーンテキストコンテンツを取得します
- [`mdast-zone`](https://github.com/syntax-tree/mdast-zone)
  — HTML コメントを範囲またはマーカーとして扱います

## 参考文献

- **unist**:
  [Universal Syntax Tree][unist].
  T. Wormer; et al.
- **Markdown**:
  [Markdown][].
  J. Gruber.
- **CommonMark**:
  [CommonMark][].
  J. MacFarlane; et al.
- **GFM**:
  [GitHub Flavored Markdown][gfm].
  GitHub.
- **HTML**:
  [HTML Standard][html],
  A. van Kesteren; et al.
  WHATWG.
- **CSSTEXT**:
  [CSS Text][css-text],
  CSS Text, E. Etemad, K. Ishii.
  W3C.
- **JavaScript**:
  [ECMAScript Language Specification][javascript].
  Ecma International.
- **YAML**:
  [YAML Ain't Markup Language][yaml],
  O. Ben-Kiki, C. Evans, I. döt Net.
- **Web IDL**:
  [Web IDL][webidl],
  C. McCormack.
  W3C.

## セキュリティ

mdast は HTML を含むことができ、HTML を表現するために使用できるため、HTML の不適切な使用が[クロスサイトスクリプティング（XSS）][xss]攻撃に晒される可能性があるように、mdast の不適切な使用も安全ではありません。
HTML に変換する際（通常は[**hast**][hast]を通じて）、常にユーザー入力に注意し、[`hast-util-santize`][sanitize]を使用して hast ツリーを安全にしてください。

## 関連項目

- [hast](https://github.com/syntax-tree/hast)
  — ハイパーテキスト抽象構文木フォーマット
- [nlcst](https://github.com/syntax-tree/nlcst)
  — 自然言語具体構文木フォーマット
- [xast](https://github.com/syntax-tree/xast)
  — 拡張可能な抽象構文木

## 貢献

開始する方法については、[`syntax-tree/.github`][health]の[`contributing.md`][contributing]を参照してください。
ヘルプを得る方法については[`support.md`][support]を参照してください。
新しいユーティリティやツールのアイデアは[`syntax-tree/ideas`][ideas]に投稿できます。

素晴らしい syntax-tree、unist、mdast、hast、xast、nlcst リソースの厳選されたリストは[awesome syntax-tree][awesome]で見つけることができます。

このプロジェクトには[行動規範][coc]があります。
このリポジトリ、組織、またはコミュニティと対話することで、あなたはその条件に従うことに同意します。

## 謝辞

このプロジェクトの初期リリースは[**@wooorm**](https://github.com/wooorm)によって作成されました。

[**@eush77**](https://github.com/eush77)の作業、アイデア、そして信じられないほど貴重なフィードバックに特別な感謝を捧げます！

以下の方々にも感謝します：
[**@anandthakker**](https://github.com/anandthakker),
[**@arobase-che**](https://github.com/arobase-che),
[**@BarryThePenguin**](https://github.com/BarryThePenguin),
[**@chinesedfan**](https://github.com/chinesedfan),
[**@ChristianMurphy**](https://github.com/ChristianMurphy),
[**@craftzdog**](https://github.com/craftzdog),
[**@d4rekanguok**](https://github.com/d4rekanguok),
[**@detj**](https://github.com/detj),
[**@dominictarr**](https://github.com/dominictarr),
[**@gkatsev**](https://github.com/gkatsev),
[**@Hamms**](https://github.com/Hamms),
[**@Hypercubed**](https://github.com/Hypercubed),
[**@ikatyang**](https://github.com/ikatyang),
[**@izumin5210**](https://github.com/izumin5210),
[**@jasonLaster**](https://github.com/jasonLaster),
[**@Justineo**](https://github.com/Justineo),
[**@justjake**](https://github.com/justjake),
[**@KyleAMathews**](https://github.com/KyleAMathews),
[**@laysent**](https://github.com/laysent),
[**@macklinu**](https://github.com/macklinu),
[**@mike-north**](https://github.com/mike-north),
[**@Murderlon**](https://github.com/Murderlon),
[**@nevik**](https://github.com/nevik),
[**@Rokt33r**](https://github.com/Rokt33r),
[**@rhysd**](https://github.com/rhysd),
[**@rubys**](https://github.com/rubys),
[**@Sarah-Seo**](https://github.com/Sarah-Seo),
[**@sethvincent**](https://github.com/sethvincent),
[**@silvenon**](https://github.com/silvenon),
[**@simov**](https://github.com/simov),
[**@staltz**](https://github.com/staltz),
[**@stefanprobst**](https://github.com/stefanprobst),
[**@tmcw**](https://github.com/tmcw),
そして [**@vhf**](https://github.com/vhf)
mdast と関連プロジェクトに貢献してくださってありがとうございます！

## ライセンス

[CC-BY-4.0][license] © [Titus Wormer][author]

<!-- 定義 -->

[health]: https://github.com/syntax-tree/.github
[contributing]: https://github.com/syntax-tree/.github/blob/HEAD/contributing.md
[support]: https://github.com/syntax-tree/.github/blob/HEAD/support.md
[coc]: https://github.com/syntax-tree/.github/blob/HEAD/code-of-conduct.md
[awesome]: https://github.com/syntax-tree/awesome-syntax-tree
[ideas]: https://github.com/syntax-tree/ideas
[license]: https://creativecommons.org/licenses/by/4.0/
[author]: https://wooorm.com
[logo]: https://raw.githubusercontent.com/syntax-tree/mdast/e6b43aa/logo.svg?sanitize=true
[releases]: https://github.com/syntax-tree/mdast/releases
[latest]: https://github.com/syntax-tree/mdast/releases/tag/5.0.0
[dfn-node]: https://github.com/syntax-tree/unist#node
[dfn-unist-parent]: https://github.com/syntax-tree/unist#parent
[dfn-unist-literal]: https://github.com/syntax-tree/unist#literal
[dfn-parent]: #parent
[dfn-literal]: #literal
[dfn-code]: #code
[dfn-inline-code]: #inlinecode
[dfn-list]: #list
[dfn-table]: #table
[dfn-break]: #break
[dfn-link-reference]: #linkreference
[dfn-image-reference]: #imagereference
[dfn-footnote-reference]: #footnotereference
[dfn-definition]: #definition
[dfn-footnote-definition]: #footnotedefinition
[term-tree]: https://github.com/syntax-tree/unist#tree
[term-child]: https://github.com/syntax-tree/unist#child
[term-sibling]: https://github.com/syntax-tree/unist#sibling
[term-root]: https://github.com/syntax-tree/unist#root
[term-head]: https://github.com/syntax-tree/unist#head
[dfn-mxn-resource]: #resource
[dfn-mxn-association]: #association
[dfn-mxn-reference]: #reference
[dfn-mxn-alternative]: #alternative
[dfn-enum-align-type]: #aligntype
[dfn-enum-reference-type]: #referencetype
[dfn-mdast-content]: #content-model
[dfn-flow-content]: #flowcontent
[dfn-frontmatter-content]: #frontmattercontent
[dfn-content]: #content
[dfn-list-content]: #listcontent
[dfn-table-content]: #tablecontent
[dfn-row-content]: #rowcontent
[dfn-phrasing-content]: #phrasingcontent
[list-of-utilities]: #list-of-utilities
[unist]: https://github.com/syntax-tree/unist
[syntax-tree]: https://github.com/syntax-tree/unist#syntax-tree
[yaml]: https://yaml.org
[html]: https://html.spec.whatwg.org/multipage/
[css-text]: https://drafts.csswg.org/css-text/
[css-left]: https://drafts.csswg.org/css-text/#valdef-text-align-left
[css-right]: https://drafts.csswg.org/css-text/#valdef-text-align-right
[css-center]: https://drafts.csswg.org/css-text/#valdef-text-align-center
[javascript]: https://www.ecma-international.org/ecma-262/9.0/index.html
[webidl]: https://heycam.github.io/webidl/
[markdown]: https://daringfireball.net/projects/markdown/
[commonmark]: https://commonmark.org
[gfm]: https://github.github.com/gfm/
[glossary]: https://github.com/syntax-tree/unist#glossary
[utilities]: https://github.com/syntax-tree/unist#list-of-utilities
[unified]: https://github.com/unifiedjs/unified
[remark]: https://github.com/remarkjs/remark
[xss]: https://en.wikipedia.org/wiki/Cross-site_scripting
[hast]: https://github.com/syntax-tree/hast
[sanitize]: https://github.com/syntax-tree/hast-util-sanitize
