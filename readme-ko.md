# ![mdast][logo]

**M**ark**d**own **A**bstract **S**yntax **T**ree.

---

**mdast**는 [구문 트리][syntax-tree]에서 마크다운을 표현하기 위한 사양입니다.
**[unist][]** 를 구현합니다.
[CommonMark][commonmark]와 [GitHub Flavored Markdown][gfm]과 같은 여러 [마크다운][markdown] 종류를 표현할 수 있습니다.

이 문서는 아직 릴리스되지 않았을 수 있습니다.
릴리스된 문서는 [releases][]를 참조하세요.
최신 릴리스 버전은 [`5.0.0`][latest]입니다.

## 목차

- [소개](#소개)
  - [이 사양의 적용 범위](#이-사양의-적용-범위)
- [타입](#타입)
- [노드 (추상)](#노드-추상)
  - [`Literal`](#literal)
  - [`Parent`](#parent)
- [노드](#노드)
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
- [믹스인](#믹스인)
  - [`Alternative`](#alternative)
  - [`Association`](#association)
  - [`Reference`](#reference)
  - [`Resource`](#resource)
- [열거형](#열거형)
  - [`referenceType`](#referencetype)
- [컨텐츠 모델](#컨텐츠-모델)
  - [`Content`](#content)
  - [`FlowContent`](#flowcontent)
  - [`ListContent`](#listcontent)
  - [`PhrasingContent`](#phrasingcontent)
- [확장](#확장)
  - [GFM](#gfm)
  - [Frontmatter](#frontmatter)
  - [MDX](#mdx)
- [용어집](#용어집)
- [유틸리티 목록](#유틸리티-목록)
- [참고 문헌](#참고-문헌)
- [보안](#보안)
- [관련 항목](#관련-항목)
- [기여](#기여)
- [감사의 말](#감사의-말)
- [라이선스](#라이선스)

## 소개

이 문서는 [마크다운][]을 [추상 구문 트리][syntax-tree]로 표현하는 형식을 정의합니다.
mdast의 개발은 2014년 7월 **[remark][remark]** 에서 시작되었으며, 이는 [unist][]가 존재하기 전이었습니다.
이 사양은 [Web IDL][webidl]과 유사한 문법으로 작성되었습니다.

### 이 사양의 적용 범위

mdast는 [unist][]를 확장하여 구문 트리를 확장하여, [유틸리티 생태계][utilities]의 이점을 얻습니다.

mdast는 [JavaScript][]와 관련이 있습니다. JavaScript에서 호환되는 구문 트리를 다루기 위한 풍부한 [유틸리티 생태계][list-of-utilities]가 있기 때문입니다.
그러나 mdast는 JavaScript에 국한되지 않으며 다른 프로그래밍 언어에서도 사용될 수 있습니다.

mdast는 [unified][] 및 [remark][] 프로젝트와 관련이 있습니다. mdast 구문 트리가 이들의 생태계 전반에 걸쳐 사용되기 때문입니다.

## 타입

TypeScript를 사용하는 경우, npm을 통해 unist 타입을 설치하여 사용할 수 있습니다:

```sh
npm install @types/mdast
```

## 노드 (추상)

### `Literal`

```idl
interface Literal <: UnistLiteral {
  value: string
}
```

**Literal** ([**UnistLiteral**][dfn-unist-literal])은 mdast에서 값을 포함하는 추상 인터페이스를 나타냅니다.

`value` 필드는 `string` 타입입니다.

### `Parent`

```idl
interface Parent <: UnistParent {
  children: [MdastContent]
}
```

**Parent** ([**UnistParent**][dfn-unist-parent])는 mdast에서 다른 노드를 포함하는 추상 인터페이스를 나타냅니다 (이를 [_자식_][term-child]이라고 합니다).

그 내용은 [**mdast 콘텐츠**][dfn-mdast-content]로만 제한됩니다.

## 노드

### `Blockquote`

```idl
interface Blockquote <: Parent {
  type: 'blockquote'
  children: [FlowContent]
}
```

**Blockquote** ([**Parent**][dfn-parent])는 다른 곳에서 인용된 섹션을 나타냅니다.

**Blockquote**는 [**흐름**][dfn-flow-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 콘텐츠 모델도 [**흐름**][dfn-flow-content] 콘텐츠입니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
> Alpha bravo charlie.
```

다음과 같이 변환됩니다:

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

**Break** ([**Node**][dfn-node])는 시(poems)나 주소에서와 같은 줄 바꿈을 나타냅니다.

**Break**는 [**구문**][dfn-phrasing-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
콘텐츠 모델이 없습니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
foo··
bar
```

다음과 같이 변환됩니다:

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

**Code** ([**Literal**][dfn-literal])는 ASCII 아트나 컴퓨터 코드와 같은 미리 형식화된 텍스트 블록을 나타냅니다.

**Code**는 [**흐름**][dfn-flow-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 내용은 `value` 필드로 표현됩니다.

이 노드는 [**구문**][dfn-phrasing-content] 콘텐츠 개념인 [**InlineCode**][dfn-inline-code]와 관련이 있습니다.

`lang` 필드가 존재할 수 있습니다.
이는 마크업되는 컴퓨터 코드의 언어를 나타냅니다.

`lang` 필드가 존재하는 경우, `meta` 필드도 존재할 수 있습니다.
이는 노드와 관련된 사용자 정의 정보를 나타냅니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
    foo()
```

다음과 같이 변환됩니다:

```js
{
  type: 'code',
  lang: null,
  meta: null,
  value: 'foo()'
}
```

그리고 다음과 같은 마크다운은:

````markdown
```js highlight-line="2"
foo();
bar();
baz();
```
````

다음과 같이 변환됩니다:

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

**Definition** ([**Node**][dfn-node])은 리소스를 나타냅니다.

**Definition**은 [**콘텐츠**][dfn-content]가 예상되는 곳에서 사용할 수 있습니다.
콘텐츠 모델이 없습니다.

**Definition**은 [**Association**][dfn-mxn-association]과 [**Resource**][dfn-mxn-resource] 믹스인을 포함합니다.

**Definition**은 [**LinkReferences**][dfn-link-reference]와 [**ImageReferences**][dfn-image-reference]와 연관되어야 합니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
[Alpha]: https://example.com
```

다음과 같이 변환됩니다:

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

**Emphasis** ([**Parent**][dfn-parent])는 그 내용의 강조를 나타냅니다.

**Emphasis**는 [**구문**][dfn-phrasing-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 콘텐츠 모델은 [**구문**][dfn-phrasing-content] 콘텐츠입니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
_alpha_ _bravo_
```

다음과 같이 변환됩니다:

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

**Heading** ([**Parent**][dfn-parent])은 섹션의 제목을 나타냅니다.

**Heading**은 [**흐름**][dfn-flow-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 콘텐츠 모델은 [**구문**][dfn-phrasing-content] 콘텐츠입니다.

`depth` 필드가 존재해야 합니다.
`1`의 값은 가장 높은 순위를 나타내고 `6`은 가장 낮은 순위를 나타냅니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
# Alpha
```

다음과 같이 변환됩니다:

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

**Html** ([**Literal**][dfn-literal])은 원시 [HTML][] 조각을 나타냅니다.

**Html**은 [**흐름**][dfn-flow-content] 또는 [**구문**][dfn-phrasing-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 내용은 `value` 필드로 표현됩니다.

**Html** 노드는 유효하거나 완전한 HTML ([\[HTML\]][html]) 구조일 필요는 없습니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
<div>
```

다음과 같이 변환됩니다:

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

**Image** ([**Node**][dfn-node])는 이미지를 나타냅니다.

**Image**는 [**구문**][dfn-phrasing-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
콘텐츠 모델은 없지만, `alt` 필드로 설명됩니다.

**Image**는 [**Resource**][dfn-mxn-resource]와 [**Alternative**][dfn-mxn-alternative] 믹스인을 포함합니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
![alpha](https://example.com/favicon.ico "bravo")
```

다음과 같이 변환됩니다:

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

**ImageReference** ([**Node**][dfn-node])는 연관을 통해 이미지를 나타내거나, 연관이 없는 경우 원본 소스를 나타냅니다.

**ImageReference**는 [**구문**][dfn-phrasing-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
콘텐츠 모델은 없지만, `alt` 필드로 설명됩니다.

**ImageReference**는 [**Reference**][dfn-mxn-reference]와 [**Alternative**][dfn-mxn-alternative] 믹스인을 포함합니다.

**ImageReference**는 [**Definition**][dfn-definition]과 연관되어야 합니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
![alpha][bravo]
```

다음과 같이 변환됩니다:

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

**InlineCode** ([**Literal**][dfn-literal])는 파일 이름, 컴퓨터 프로그램 또는 컴퓨터가 구문 분석할 수 있는 모든 것과 같은 컴퓨터 코드 조각을 나타냅니다.

**InlineCode**는 [**구문**][dfn-phrasing-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 내용은 `value` 필드로 표현됩니다.

이 노드는 [**흐름**][dfn-flow-content] 콘텐츠 개념인 [**Code**][dfn-code]와 관련이 있습니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
`foo()`
```

다음과 같이 변환됩니다:

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

**Link** ([**Parent**][dfn-parent])는 하이퍼링크를 나타냅니다.

**Link**는 [**구문**][dfn-phrasing-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 콘텐츠 모델도 [**구문**][dfn-phrasing-content] 콘텐츠입니다.

**Link**는 [**Resource**][dfn-mxn-resource] 믹스인을 포함합니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
[alpha](https://example.com "bravo")
```

다음과 같이 변환됩니다:

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

**LinkReference** ([**Parent**][dfn-parent])는 연관을 통해 하이퍼링크를 나타내거나, 연관이 없는 경우 원본 소스를 나타냅니다.

**LinkReference**는 [**구문**][dfn-phrasing-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 콘텐츠 모델도 [**구문**][dfn-phrasing-content] 콘텐츠입니다.

**LinkReference**는 [**Reference**][dfn-mxn-reference] 믹스인을 포함합니다.

**LinkReferences**는 [**Definition**][dfn-definition]과 연관되어야 합니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
[alpha][Bravo]
```

다음과 같이 변환됩니다:

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

**List** ([**Parent**][dfn-parent])는 항목의 목록을 나타냅니다.

**List**는 [**흐름**][dfn-flow-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 콘텐츠 모델은 [**목록**][dfn-list-content] 콘텐츠입니다.

`ordered` 필드가 존재할 수 있습니다.
이는 `true`인 경우 항목들이 의도적으로 순서가 지정되었음을 나타냅니다. 또는 `false`이거나 존재하지 않는 경우 항목의 순서가 중요하지 않음을 나타냅니다.

`start` 필드가 존재할 수 있습니다.
이는 `ordered` 필드가 `true`일 때, 목록의 시작 번호를 나타냅니다.

`spread` 필드가 존재할 수 있습니다.
이는 하나 이상의 자식이 [형제][term-sibling]와 빈 줄로 구분되어 있음을 나타냅니다 (`true`인 경우), 또는 그렇지 않음을 나타냅니다 (`false`이거나 존재하지 않는 경우).

예를 들어, 다음과 같은 마크다운은:

```markdown
1. foo
```

다음과 같이 변환됩니다:

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

**ListItem** ([**Parent**][dfn-parent])는 [**List**][dfn-list]의 항목을 나타냅니다.

**ListItem**은 [**목록**][dfn-list-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 콘텐츠 모델은 [**흐름**][dfn-flow-content] 콘텐츠입니다.

`spread` 필드가 존재할 수 있습니다.
이는 `true`인 경우 항목이 빈 줄로 구분된 두 개 이상의 [_자식_][term-child]을 포함하고 있음을 나타냅니다. 또는 `false`이거나 존재하지 않는 경우 그렇지 않음을 나타냅니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
- bar
```

다음과 같이 변환됩니다:

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

**Paragraph** ([**Parent**][dfn-parent])는 특정 주제나 아이디어를 다루는 단락의 단위를 나타냅니다.

**Paragraph**는 [**콘텐츠**][dfn-content]가 예상되는 곳에서 사용할 수 있습니다.
그 콘텐츠 모델은 [**구문**][dfn-phrasing-content] 콘텐츠입니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
Alpha bravo charlie.
```

다음과 같이 변환됩니다:

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

**Root** ([**Parent**][dfn-parent])는 문서를 나타냅니다.

**Root**는 [_트리_][term-tree]의 [_루트_][term-root]로 사용될 수 있으며, 절대 [_자식_][term-child]으로 사용될 수 없습니다.
그 콘텐츠 모델은 [**흐름**][dfn-flow-content] 콘텐츠로 제한되지 않지만, 대신 모든 [**mdast 콘텐츠**][dfn-mdast-content]를 포함할 수 있으며, 모든 콘텐츠가 동일한 카테고리여야 한다는 제한이 있습니다.

### `Strong`

```idl
interface Strong <: Parent {
  type: 'strong'
  children: [PhrasingContent]
}
```

**Strong** ([**Parent**][dfn-parent])는 그 내용에 대한 강한 중요성, 심각성 또는 긴급성을 나타냅니다.

**Strong**은 [**구문**][dfn-phrasing-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 콘텐츠 모델은 [**구문**][dfn-phrasing-content] 콘텐츠입니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
**alpha** **bravo**
```

다음과 같이 변환됩니다:

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

**Text** ([**Literal**][dfn-literal])는 단순히 텍스트인 모든 것을 나타냅니다.

**Text**는 [**구문**][dfn-phrasing-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 내용은 `value` 필드로 표현됩니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
Alpha bravo charlie.
```

다음과 같이 변환됩니다:

```js
{type: 'text', value: 'Alpha bravo charlie.'}
```

### `ThematicBreak`

```idl
interface ThematicBreak <: Node {
  type: 'thematicBreak'
}
```

**ThematicBreak** ([**Node**][dfn-node])는 이야기의 장면 전환, 다른 주제로의 전환 또는 새로운 문서와 같은 주제별 구분을 나타냅니다.

**ThematicBreak**는 [**흐름**][dfn-flow-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
콘텐츠 모델이 없습니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
---
```

다음과 같이 변환됩니다:

```js
{
  type: "thematicBreak";
}
```

## 믹스인

### `Alternative`

```idl
interface mixin Alternative {
  alt: string?
}
```

**Alternative**는 대체 내용이 있는 노드를 나타냅니다.

`alt` 필드가 존재해야 합니다.
이는 노드를 의도한 대로 표현할 수 없는 환경에 대한 동등한 내용을 나타냅니다.

### `Association`

```idl
interface mixin Association {
  identifier: string
  label: string?
}
```

**Association**은 한 노드에서 다른 노드로의 내부 관계를 나타냅니다.

`identifier` 필드가 존재해야 합니다.
이는 다른 노드와 일치할 수 있습니다.
`identifier`는 소스 값입니다: 문자 이스케이프와 문자 참조는 _파싱되지 않습니다_. 그 값은 정규화되어야 합니다.

`label` 필드가 존재할 수 있습니다.
`label`은 문자열 값입니다: 링크의 `title`이나 코드의 `lang`과 마찬가지로 작동합니다: 문자 이스케이프와 문자 참조가 파싱됩니다.

값을 정규화하려면 마크다운 공백(`[\t\n\r ]+`)을 공백으로 축소하고, 선택적 초기 및/또는 최종 공백을 제거하며, 대소문자 접기를 수행합니다.

`identifier`의 값(또는 `identifier`가 없는 경우 정규화된 `label`)이 고유 식별자여야 하는지 여부는 **Association**을 포함하는 노드의 유형에 따라 다릅니다.
예를 들어, [**Definition**][dfn-definition]에서는 고유해야 하지만, 여러 [**LinkReference**][dfn-link-reference]는 하나의 정의와 연관되기 위해 고유하지 않을 수 있습니다.

### `Reference`

```idl
interface mixin Reference {
  referenceType: string
}

Reference includes Association
```

**Reference**는 다른 노드와 [**연관된**][dfn-mxn-association] 마커를 나타냅니다.

`referenceType` 필드가 존재해야 합니다.
그 값은 [**referenceType**][dfn-enum-reference-type]이어야 합니다.
이는 참조의 명시성을 나타냅니다.

### `Resource`

```idl
interface mixin Resource {
  url: string
  title: string?
}
```

**Resource**는 리소스에 대한 참조를 나타냅니다.

`url` 필드가 존재해야 합니다.
이는 참조된 리소스에 대한 URL을 나타냅니다.

`title` 필드가 존재할 수 있습니다.
이는 리소스에 대한 조언 정보를 나타내며, 툴팁에 적합할 수 있습니다.

## 열거형

### `referenceType`

```idl
enum referenceType {
  'shortcut' | 'collapsed' | 'full'
}
```

**referenceType**은 참조의 명시성을 나타냅니다.

- **shortcut**: 참조가 암시적이며, 식별자가 그 내용에서 추론됩니다.
- **collapsed**: 참조가 명시적이며, 식별자가 그 내용에서 추론됩니다.
- **full**: 참조가 명시적이며, 식별자가 명시적으로 설정됩니다.

## 컨텐츠 모델

```idl
type MdastContent = FlowContent | ListContent | PhrasingContent
```

mdast의 각 노드는 유사한 특성을 가진 노드들을 그룹화하는 하나 이상의 **Content** 카테고리에 속합니다.

### `Content`

```idl
type Content = Definition | Paragraph
```

**Content**는 정의와 단락을 형성하는 텍스트의 실행을 나타냅니다.

### `FlowContent`

```idl
type FlowContent =
  Blockquote | Code | Heading | Html | List | ThematicBreak | Content
```

**Flow** 콘텐츠는 문서의 섹션을 나타냅니다.

### `ListContent`

```idl
type ListContent = ListItem
```

**List** 콘텐츠는 목록의 항목을 나타냅니다.

### `PhrasingContent`

```idl
type PhrasingContent = Break | Emphasis | Html | Image | ImageReference
  | InlineCode | Link | LinkReference | Strong | Text
```

**Phrasing** 콘텐츠는 문서의 텍스트와 그 마크업을 나타냅니다.

## 확장

마크다운 구문은 종종 확장됩니다.
이 명세의 목표는 모든 가능한 확장을 나열하는 것이 아닙니다.
그러나 자주 사용되는 확장의 짧은 목록이 아래에 나와 있습니다.

### GFM

다음 인터페이스들은 [GitHub Flavored Markdown][gfm]에서 찾을 수 있습니다.

#### `Delete`

```idl
interface Delete <: Parent {
  type: 'delete'
  children: [PhrasingContent]
}
```

**Delete** ([**Parent**][dfn-parent])는 더 이상 정확하지 않거나 더 이상 관련이 없는 내용을 나타냅니다.

**Delete**는 [**구문**][dfn-phrasing-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 콘텐츠 모델은 [**구문**][dfn-phrasing-content] 콘텐츠입니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
~~alpha~~
```

다음과 같이 변환됩니다:

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

GFM에서는 `checked` 필드가 존재할 수 있습니다.
이는 항목이 완료되었는지(`true`일 때), 완료되지 않았는지(`false`일 때), 또는 결정되지 않았거나 해당 사항이 없는지(`null`이거나 존재하지 않을 때)를 나타냅니다.

#### `FootnoteDefinition`

```idl
interface FootnoteDefinition <: Parent {
  type: 'footnoteDefinition'
  children: [FlowContent]
}

FootnoteDefinition includes Association
```

**FootnoteDefinition** ([**Parent**][dfn-parent])는 문서의 흐름 외부에 있는 문서와 관련된 내용을 나타냅니다.

**FootnoteDefinition**은 [**흐름**][dfn-flow-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 콘텐츠 모델도 [**흐름**][dfn-flow-content] 콘텐츠입니다.

**FootnoteDefinition**은 [**Association**][dfn-mxn-association] 믹스인을 포함합니다.

**FootnoteDefinition**은 [**FootnoteReferences**][dfn-footnote-reference]와 연관되어야 합니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
[^alpha]: bravo and charlie.
```

다음과 같이 변환됩니다:

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

**FootnoteReference** ([**Node**][dfn-node])는 연관을 통한 마커를 나타냅니다.

**FootnoteReference**는 [**구문**][dfn-phrasing-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
콘텐츠 모델이 없습니다.

**FootnoteReference**는 [**Association**][dfn-mxn-association] 믹스인을 포함합니다.

**FootnoteReference**는 [**FootnoteDefinition**][dfn-footnote-definition]과 연관되어야 합니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
[^alpha]
```

다음과 같이 변환됩니다:

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

**Table** ([**Parent**][dfn-parent])는 2차원 데이터를 나타냅니다.

**Table**은 [**흐름**][dfn-flow-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 콘텐츠 모델은 [**테이블**][dfn-table-content] 콘텐츠입니다.

노드의 [_헤더_][term-head]는 열의 레이블을 나타냅니다.

`align` 필드가 존재할 수 있습니다.
존재하는 경우, [**alignType**][dfn-enum-align-type]의 목록이어야 합니다.
이는 열의 셀들이 어떻게 정렬되는지를 나타냅니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
| foo | bar |
| :-- | :-: |
| baz | qux |
```

다음과 같이 변환됩니다:

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

**TableCell** ([**Parent**][dfn-parent])는 [**Table**][dfn-table]의 헤더 셀을 나타냅니다, 만약 그 부모가 [_헤더_][term-head]인 경우, 그렇지 않으면 데이터 셀을 나타냅니다.

**TableCell**은 [**행**][dfn-row-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 콘텐츠 모델은 [**Break**][dfn-break] 노드를 제외한 [**구문**][dfn-phrasing-content] 콘텐츠입니다.

예시는 [**Table**][dfn-table]을 참조하세요.

#### `TableRow`

```idl
interface TableRow <: Parent {
  type: 'tableRow'
  children: [RowContent]
}
```

**TableRow** ([**Parent**][dfn-parent])는 테이블의 셀 행을 나타냅니다.

**TableRow**는 [**테이블**][dfn-table-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 콘텐츠 모델은 [**행**][dfn-row-content] 콘텐츠입니다.

노드가 [_헤더_][term-head]인 경우, 부모 [**Table**][dfn-table]의 열 레이블을 나타냅니다.

예시는 [**Table**][dfn-table]을 참조하세요.

#### `alignType`

```idl
enum alignType {
  'left' | 'right' | 'center' | null
}
```

**alignType**은 구문 콘텐츠가 어떻게 정렬되는지를 나타냅니다 ([\[CSSTEXT\]][css-text]).

- **`'left'`**: `text-align` CSS 속성의 [`left`][css-left] 값 참조
- **`'right'`**: `text-align` CSS 속성의 [`right`][css-right] 값 참조
- **`'center'`**: `text-align` CSS 속성의 [`center`][css-center] 값 참조
- **`null`**: 구문 콘텐츠가 호스트 환경에 의해 정의된 대로 정렬됨

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

**Row** 콘텐츠는 행의 셀을 나타냅니다.

#### `TableContent`

```idl
type TableContent = TableRow
```

**Table** 콘텐츠는 테이블의 행을 나타냅니다.

### Frontmatter

다음 인터페이스들은 YAML에서 찾을 수 있습니다.

#### `Yaml`

```idl
interface Yaml <: Literal {
  type: 'yaml'
}
```

**Yaml** ([**Literal**][dfn-literal])은 YAML ([\[YAML\]][yaml]) 데이터 직렬화 언어로 작성된 문서의 메타데이터 컬렉션을 나타냅니다.

**Yaml**은 [**frontmatter**][dfn-frontmatter-content] 콘텐츠가 예상되는 곳에서 사용할 수 있습니다.
그 내용은 `value` 필드로 표현됩니다.

예를 들어, 다음과 같은 마크다운은:

```markdown
---
foo: bar
---
```

다음과 같이 변환됩니다:

```js
{type: 'yaml', value: 'foo: bar'}
```

#### `FrontmatterContent`

```idl
type FrontmatterContent = Yaml
```

**Frontmatter** 콘텐츠는 문서에 대한 대역 외 정보를 나타냅니다.

frontmatter가 존재하는 경우, [_트리_][term-tree]에서 하나의 노드로 제한되어야 하며, [_헤더_][term-head]로만 존재할 수 있습니다.

#### `FlowContent` (frontmatter)

```idl
type FlowContentFrontmatter = FrontmatterContent | FlowContent
```

### MDX

[`remark-mdx`](https://mdxjs.com/packages/remark-mdx/#syntax-tree)를 참조하세요.

## 용어집

[unist 용어집][glossary]을 참조하세요.

## 유틸리티 목록

더 많은 유틸리티는 [unist 유틸리티 목록][utilities]을 참조하세요.

<!--lint disable list-item-spacing-->

- [`mdast-add-list-metadata`](https://gitlab.com/staltz/mdast-add-list-metadata)
  — `list`와 `listItem` 노드의 메타데이터를 강화합니다
- [`mdast-util-assert`](https://github.com/syntax-tree/mdast-util-assert)
  — 노드를 확인합니다
- [`mdast-builder`](https://github.com/mike-north/mdast-builder)
  — 조합 가능한 함수로 mdast 구조를 구축합니다
- [`mdast-comment-marker`](https://github.com/syntax-tree/mdast-comment-marker)
  — 주석 마커를 파싱합니다
- [`mdast-util-compact`](https://github.com/syntax-tree/mdast-util-compact)
  — 트리를 압축합니다
- [`mdast-util-definitions`](https://github.com/syntax-tree/mdast-util-definitions)
  — 정의 노드를 찾습니다
- [`mdast-util-directive`](https://github.com/syntax-tree/mdast-util-directive)
  — 지시문을 파싱하고 직렬화합니다
- [`mdast-util-find-and-replace`](https://github.com/syntax-tree/mdast-util-find-and-replace)
  — 텍스트를 찾아 바꿉니다
- [`mdast-flatten-image-paragraphs`](https://gitlab.com/staltz/mdast-flatten-image-paragraphs)
  — `paragraph`와 `image`를 하나의 `image` 노드로 평탄화합니다
- [`mdast-flatten-listitem-paragraphs`](https://gitlab.com/staltz/mdast-flatten-listitem-paragraphs)
  — `listItem`과 (중첩된) 단락을 하나의 listItem 노드로 평탄화합니다
- [`mdast-flatten-nested-lists`](https://gitlab.com/staltz/mdast-flatten-nested-lists)
  — 목록 안의 목록을 피하도록 트리를 변환합니다
- [`mdast-util-from-adf`](https://github.com/bitcrowd/mdast-util-from-adf)
  — Atlassian Document Format (ADF)에서 mdast 구문 트리를 구축합니다
- [`mdast-util-from-markdown`](https://github.com/syntax-tree/mdast-util-from-markdown)

  — 마크다운을 파싱합니다

- [`mdast-util-frontmatter`](https://github.com/syntax-tree/mdast-util-frontmatter)
  — frontmatter를 파싱하고 직렬화합니다
- [`mdast-util-gfm`](https://github.com/syntax-tree/mdast-util-gfm)
  — GFM을 파싱하고 직렬화합니다
- [`mdast-util-gfm-autolink-literal`](https://github.com/syntax-tree/mdast-util-gfm-autolink-literal)
  — GFM 자동 링크 리터럴을 파싱하고 직렬화합니다
- [`mdast-util-gfm-footnote`](https://github.com/syntax-tree/mdast-util-gfm-footnote)
  — GFM 각주를 파싱하고 직렬화합니다
- [`mdast-util-gfm-strikethrough`](https://github.com/syntax-tree/mdast-util-gfm-strikethrough)
  — GFM 취소선을 파싱하고 직렬화합니다
- [`mdast-util-gfm-table`](https://github.com/syntax-tree/mdast-util-gfm-table)
  — GFM 테이블을 파싱하고 직렬화합니다
- [`mdast-util-gfm-task-list-item`](https://github.com/syntax-tree/mdast-util-gfm-task-list-item)
  — GFM 작업 목록 항목을 파싱하고 직렬화합니다
- [`mdast-util-gridtables`](https://github.com/syntax-tree/mdast-util-gridtables)
  — 격자 테이블을 파싱하고 직렬화합니다
- [`mdast-util-heading-range`](https://github.com/syntax-tree/mdast-util-heading-range)
  — 마크다운 제목을 범위로 처리합니다
- [`mdast-util-heading-style`](https://github.com/syntax-tree/mdast-util-heading-style)
  — 제목 노드의 스타일을 가져옵니다
- [`mdast-util-hidden`](https://github.com/Xunnamius/unified-utils/tree/main/packages/mdast-util-hidden)
  — 노드가 변환기에 의해 보이지 않도록 합니다
- [`mdast-util-math`](https://github.com/syntax-tree/mdast-util-math)
  — 수학 표현을 파싱하고 직렬화합니다
- [`mdast-util-mdx`](https://github.com/syntax-tree/mdast-util-mdx)
  — MDX를 파싱하고 직렬화합니다
- [`mdast-util-mdx-expression`](https://github.com/syntax-tree/mdast-util-mdx-expression)
  — MDX 표현식을 파싱하고 직렬화합니다
- [`mdast-util-mdx-jsx`](https://github.com/syntax-tree/mdast-util-mdx-jsx)
  — MDX JSX를 파싱하고 직렬화합니다
- [`mdast-util-mdxjs-esm`](https://github.com/syntax-tree/mdast-util-mdxjs-esm)
  — MDX ESM을 파싱하고 직렬화합니다
- [`mdast-move-images-to-root`](https://gitlab.com/staltz/mdast-move-images-to-root)
  — 이미지 노드를 트리 위로 이동하여 루트의 직접 자식이 되도록 합니다
- [`mdast-normalize-headings`](https://github.com/syntax-tree/mdast-normalize-headings)
  — 문서에 최상위 제목이 최대 하나만 있도록 합니다
- [`mdast-util-phrasing`](https://github.com/syntax-tree/mdast-util-phrasing)
  — 노드가 구문 내용인지 확인합니다
- [`mdast-squeeze-paragraphs`](https://github.com/syntax-tree/mdast-squeeze-paragraphs)
  — 빈 단락을 제거합니다
- [`mdast-util-toc`](https://github.com/syntax-tree/mdast-util-toc)
  — 트리에서 목차를 생성합니다
- [`mdast-util-to-hast`](https://github.com/syntax-tree/mdast-util-to-hast)
  — hast로 변환합니다
- [`mdast-util-to-markdown`](https://github.com/syntax-tree/mdast-util-to-markdown)
  — 마크다운을 직렬화합니다
- [`mdast-util-to-nlcst`](https://github.com/syntax-tree/mdast-util-to-nlcst)
  — nlcst로 변환합니다
- [`mdast-util-to-string`](https://github.com/syntax-tree/mdast-util-to-string)
  — 노드의 일반 텍스트 내용을 가져옵니다
- [`mdast-zone`](https://github.com/syntax-tree/mdast-zone)
  — HTML 주석을 범위나 마커로 처리합니다

## 참고 문헌

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

## 보안

mdast는 HTML을 포함할 수 있고 HTML을 표현하는 데 사용될 수 있으므로, HTML의 부적절한 사용이 [크로스 사이트 스크립팅 (XSS)][xss] 공격에 노출될 수 있듯이 mdast의 부적절한 사용도 안전하지 않습니다.
HTML로 변환할 때 (일반적으로 [**hast**][hast]를 통해), 항상 사용자 입력에 주의하고 [`hast-util-santize`][sanitize]를 사용하여 hast 트리를 안전하게 만드세요.

## 관련 항목

- [hast](https://github.com/syntax-tree/hast)
  — 하이퍼텍스트 추상 구문 트리 형식
- [nlcst](https://github.com/syntax-tree/nlcst)
  — 자연어 구체 구문 트리 형식
- [xast](https://github.com/syntax-tree/xast)
  — 확장 가능한 추상 구문 트리

## 기여

시작하는 방법은 [`syntax-tree/.github`][health]의 [`contributing.md`][contributing]를 참조하세요.
도움을 받는 방법은 [`support.md`][support]를 참조하세요.
새로운 유틸리티와 도구에 대한 아이디어는 [`syntax-tree/ideas`][ideas]에 게시할 수 있습니다.

멋진 syntax-tree, unist, mdast, hast, xast, nlcst 리소스의 큐레이션된 목록은 [awesome syntax-tree][awesome]에서 찾을 수 있습니다.

이 프로젝트에는 [행동 강령][coc]이 있습니다.
이 저장소, 조직 또는 커뮤니티와 상호 작용함으로써 귀하는 해당 약관을 준수하는 것에 동의합니다.

## 감사의 말

이 프로젝트의 초기 릴리스는 [**@wooorm**](https://github.com/wooorm)에 의해 작성되었습니다.

[**@eush77**](https://github.com/eush77)의 작업, 아이디어, 그리고 믿을 수 없이 귀중한 피드백에 특별히 감사드립니다!

다음 분들께도 감사드립니다:
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
그리고 [**@vhf**](https://github.com/vhf)
mdast와 관련 프로젝트에 기여해 주셔서 감사합니다!

## 라이선스

[CC-BY-4.0][license] © [Titus Wormer][author]

<!-- 정의 -->

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
