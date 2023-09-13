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
  * prosemirror-model은 에디터의 문서 모델, 즉 에디터의 콘텐츠를 설명하는 데 사용되는 데이터 구조를 정의한다.
  * prosemirror-state는 선택 항목을 포함한 편집기의 전체 상태를 설명하는 데이터 구조와 한 상태에서 다음 상태로 이동하기 위한 트랜잭션 시스템을 제공한다.
  * prosemirror-view는 지정된 편집기 상태를 브라우저에서 편집 가능한 요소로 표시하고 해당 요소와의 사용자 상호 작용을 처리하는 사용자 인터페이스 구성 요소를 구현한다.
  * prosemirror-transform에는 상태 모듈의 트랜잭션의 기반이 되는 기록 및 재생이 가능한 방식으로 문서를 수정하고 실행 취소 기록과 공동 편집을 가능하게 하는 기능이 포함되어 있다.
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