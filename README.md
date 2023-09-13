## ProseMirror 학습

https://prosemirror.net/examples/ 참고

## 실행 방법

```shell
pnpm install
pnpm watch
```

## ProseMirror 가이드

### 소개

* ProseMirror 의 주요 원칙은 코드가 document와 document에서 일어나는 모든 일을 제어하는 것이다.
* core 라이브러리는 쉬운 drop-in-component 형식이 아니다. 미래에 ProseMirror 를 사용해서 drop-in-component 형식의 편집기를 배포할 것이라는 생각으로 모듈성과 사용자 정의 기능에 초첨을 두고 있다.
* 필수 모듈은 다음과 같다.
  * prosemirror-model은 에디터의 document 모델, 즉 에디터의 콘텐츠를 설명하는 데 사용되는 데이터 구조를 정의한다.
  * prosemirror-state는 선택 항목을 포함한 편집기의 전체 상태를 설명하는 데이터 구조와 한 상태에서 다음 상태로 이동하기 위한 트랜잭션 시스템을 제공한다.
  * prosemirror-view는 지정된 편집기 상태를 브라우저에서 편집 가능한 요소로 표시하고 해당 요소와의 사용자 상호 작용을 처리하는 사용자 인터페이스 구성 요소를 구현한다.
  * prosemirror-transform에는 상태 모듈의 트랜잭션의 기반이 되는 기록 및 재생이 가능한 방식으로 document를 수정하고 실행 취소 기록과 공동 편집을 가능하게 하는 기능이 포함되어 있다.
* example

  ```ts
  import { schema } from "prosemirror-schema-basic";
  import { EditorState } from "prosemirror-state";
  import { EditorView } from "prosemirror-view";

  /**
   * NOTE
   * ProseMirror에서는 document가 준수하는 스키마를 지정해야 하므로 가장 먼저 하는 일은 기본 스키마가 포함된 모듈을 가져오는 것이다.
   * 해당 스키마는 상태를 생성하는 데 사용되며, 이는 스키마를 준수하는 빈 document와 해당 document 시작 시 기본 선택 항목을 생성한다.
   */
  let state = EditorState.create({ schema });

  /**
   * NOTE
   * 마지막으로 상태에 대한 뷰가 생성되고 document.body에 추가된다.
   * 이렇게 하면 상태의 document 가 편집 가능한 DOM 노드로 렌더링 되고 
   * 사용자가 입력할 때마다 상태 트랜잭션이 생성된다.
   */
  let view = new EditorView(document.body, { state });
  ```

### Transactions

```ts
// (Imports omitted)

/**
 * NOTE
 * 스키마를 준수하는 상태 생성
 */
let state = EditorState.create({ schema });

/**
 * NOTE
 * document 가 편집 가능한 DOM 노드 생성 및 document.body 에 추가
 * 사용자가 입력할 때마다 트랜잭션 발생
 */
let view = new EditorView(document.body, {
  state,
  dispatchTransaction(transaction) {
    console.log(
      "Document size went from",
      transaction.before.content.size,
      "to",
      transaction.doc.content.size,
    );

    /**
     * NOTE
     * 새 상태 생성
     */
    const newState = view.state.apply(transaction);

    /**
     * NOTE
     * 뷰 업데이트
     */
    view.updateState(newState);
  },
});
```

사용자가 뷰에 입력하거나 다른 방식으로 상호 작용하면 상태 트랜잭션'이 생성된다. 변경할 때마다 상태 변경 사항을 설명하는 트랜잭션이 생성되며, 이 트랜잭션을 적용하여 새 상태를 만든 다음 뷰를 업데이트하는 데 사용할 수 있다.

기본적으로 이 모든 작업은 은밀하게 이루어지지만 플러그인을 작성하거나 뷰를 구성하여 연결할 수 있다. 예를 들어, 이 코드는 트랜잭션이 생성될 때마다 호출되는 dispatchTransaction 프로퍼티를 추가한다.

### Plugins
플러그인은 편집기와 편집기 상태의 동작을 다양한 방식으로 확장하는 데 사용된다.

이 두 플러그인을 에디터에 추가하여 실행 취소/다시 실행 기능을 추가해 보자.

```ts
// (Omitted repeated imports)
import { undo, redo, history } from "prosemirror-history";
import { keymap } from "prosemirror-keymap";

const state = EditorState.create({
  schema,
  plugins: [
    /**
     * NOTE
     *  트랜잭션을 관찰하고 역으로 저장하여 취소 기록을 구현하는 히스토리 플러그인
     */
    history(), 
    /**
     * NOTE
     * 키보드 입력에 바인딩하는 키맵 플러그인
     */
    keymap({ "Mod-z": undo, "Mod-y": redo })
  ],
});

const view = new EditorView(document.body, { state });
```

## Commands

이전 예제에서 키에 바인딩된 undo 및 redo 값은 명령이라는 특별한 종류의 함수이다. 대부분의 편집 작업은 키에 바인딩되거나 메뉴에 연결되거나 사용자에게 표시될 수 있는 명령으로 작성된다.

prosemirror-commands 패키지는 편집기에서 입력 및 삭제와 같은 작업이 예상되는 작업을 수행하도록 활성화하려는 최소 키맵과 함께 여러 가지 기본 편집 명령을 제공한다.

```ts
// (Omitted repeated imports)
import { baseKeymap } from "prosemirror-commands";

let state = EditorState.create({
  schema,
  plugins: [
    history(),
    keymap({"Mod-z": undo, "Mod-y": redo}),
    keymap(baseKeymap)
  ]
})

let view = new EditorView(document.body, { state })
```

[prosemirror-example-setup](https://github.com/prosemirror/prosemirror-example-setup) 해당 프로젝트를 살펴보면 도움이 된다.

