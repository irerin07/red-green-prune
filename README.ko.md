# Red Green Prune

**중복 테스트 없는 테스트 우선 개발.**

[English](README.md)

코딩 에이전트는 종종 구현부터 시작하고 나중에 테스트를 추가하면서도
TDD를 했다고 말합니다. 같은 동작을 보호하는 테스트를 여러 개 추가하기도
합니다.

Red Green Prune은 작업 흐름을 다음과 같이 집중시킵니다.

```text
ALIGN -> RED -> GREEN -> REFACTOR -> PRUNE
```

> 프로덕션 코드를 수정하기 전에 관련 테스트 실패를 관찰하고, 실패한
> 테스트 범위와 사용자 요청에 포함된 동작만 구현하며, 서로 다른 보호
> 가치가 있는 테스트만 유지합니다.

## 적용 전 / 적용 후

사용자 이름을 대소문자 구분 없이 조회하도록 요청했다고 가정해 봅시다.

Red Green Prune이 없다면 에이전트는 기능부터 구현하고 그 뒤 같은 의미의
테스트를 여러 개 추가할 수 있습니다.

Red Green Prune을 사용하면 다음 순서로 작업합니다.

```text
ALIGN     조회 경로와 가장 가까운 테스트를 읽습니다.
RED       가장 가까운 테스트를 확장하고 누락된 동작의 실패를 관찰합니다.
GREEN     비교 로직을 변경하고 테스트를 다시 실행합니다.
REFACTOR  현재 작업에 필요하지 않으면 아무것도 변경하지 않습니다.
PRUNE     서로 다른 보호는 유지하고, 같은 의미의 새 테스트는 합치거나 생략합니다.
```

하나의 관찰 가능한 계약을 함께 설명한다면 여러 assertion이나 case가 같은
테스트 범위에 포함될 수 있습니다. Red Green Prune은 matcher마다 별도의
RED를 요구하지 않습니다.

## 모듈형 스킬

현재 작업에 필요한 책임만 로드하도록 스킬을 분리했습니다.

| 스킬 | 책임 | 사용 대상 |
| --- | --- | --- |
| `red-green-prune` | 얇은 진입점과 라우팅 | 전체 작업 과정 |
| `test-first-cycle` | 프로덕션 변경 전에 테스트 범위 실패 확인 | 기능, 수정, API, 계산, 검증 |
| `minimal-change` | 구현을 실패한 범위와 요청 안에 유지 | GREEN, REFACTOR, 비동작 변경 |
| `test-prune` | 중복 테스트 근거와 삭제 안전성 | 겹치는 새 테스트 또는 명시적 테스트 정리 |

일반적인 동작 변경에는 `test-first-cycle`과 `minimal-change`를 사용합니다.
`test-prune`은 테스트가 겹칠 가능성이 있거나 정리를 요청받았을 때만
로드합니다.

## 작동 방식

### ALIGN과 RED

영향받는 코드와 가장 가까운 테스트를 읽습니다. 요청된 동작을 표현하는
가장 작은 테스트 범위를 선택하거나 작성하고, 프로덕션 코드를 수정하기
전에 실행하여 해당 동작이 없기 때문에 실패하는지 확인합니다.

setup, 문법, flaky, 관련 없는 기존 실패는 RED가 아닙니다. 유효한 RED를
관찰할 수 없으면 TDD라고 주장하지 않고 한계를 밝힙니다.

### GREEN과 REFACTOR

실패한 테스트 범위가 단언하고 사용자가 요청한 동작만 구현합니다. 하나의
관찰 가능한 계약에 포함된 모든 assertion은 같은 GREEN에서 만족시킬 수
있습니다. focused test와 관련된 기존 테스트를 실행합니다.

테스트가 통과하는 동안 현재 변경에 필요할 때만 리팩터링합니다. 요청하지
않은 의존성, 설정, 계층, 영속성, endpoint, extension point, 관련 없는
정리를 추가하지 않습니다.

### PRUNE

현재 작업에서 생긴 중복 테스트를 방지합니다. 서로 다른 동작, 경계,
동등 클래스, 회귀, 보안 규칙, 계약, 실패 형태를 보호하는 테스트는
유지합니다.

## 안전하게 줄이기

Red Green Prune은 새로운 테스트 비대화를 방지합니다. 기존 테스트 모음을
자율적으로 정리하지 않습니다.

일반적인 기능 구현이나 버그 수정 중에는 기존 테스트를 삭제하거나,
비활성화하거나, 약화해서는 안 됩니다. 기존 중복 의심 항목은 별도로
보고합니다. 제거하려면 명시적인 정리 요청이 필요하며, 위험도가 높은
테스트는 근거와 사용자 승인도 필요합니다.

## 설치

### Codex

```bash
codex plugin marketplace add irerin07/red-green-prune
codex plugin add red-green-prune@red-green-prune
```

새 Codex 세션을 시작한 뒤 전체 작업 과정을 호출합니다.

```text
$red-green-prune 대소문자를 구분하지 않는 사용자 이름 조회를 구현해줘
```

Codex는 작업에 따라 개별 스킬을 자동으로 활성화할 수도 있습니다.

### Claude Code

다음 명령을 별도의 프롬프트로 각각 전송합니다.

```text
/plugin marketplace add irerin07/red-green-prune
```

```text
/plugin install red-green-prune@red-green-prune
```

새 세션을 시작한 뒤 전체 작업 과정을 호출합니다.

```text
/red-green-prune:red-green-prune 대소문자를 구분하지 않는 사용자 이름 조회를 구현해줘
```

책임 하나만 직접 호출할 수도 있습니다.

```text
/red-green-prune:test-first-cycle
/red-green-prune:minimal-change
/red-green-prune:test-prune
```

플러그인을 업데이트했다면 최신 스킬을 발견하도록 새 세션을 시작하세요.

## 하지 않는 것

- matcher나 assertion마다 별도의 RED를 요구하지 않습니다.
- 문서, 포매팅, 생성된 파일, 동작을 보존하는 기계적 변경에는 RED를
  요구하지 않습니다.
- 기존 저장소에는 요청 없이 테스트 프레임워크를 만들지 않습니다. 새
  프로젝트에는 최소한의 표준 테스트 환경을 구성합니다.
- 일반적인 구현 작업 중 기존 테스트 커버리지를 삭제하지 않습니다.
- 실패한 테스트 범위와 사용자 요청 밖의 동작을 구현하지 않습니다.
- 프로덕션 변경 전에 RED를 관찰하지 않았다면 TDD라고 주장하지 않습니다.

스킬 정의는 다음 파일에 있습니다.

- [`skills/red-green-prune/SKILL.md`](skills/red-green-prune/SKILL.md)
- [`skills/test-first-cycle/SKILL.md`](skills/test-first-cycle/SKILL.md)
- [`skills/minimal-change/SKILL.md`](skills/minimal-change/SKILL.md)
- [`skills/test-prune/SKILL.md`](skills/test-prune/SKILL.md)

## 현재 상태

규칙은 기록된 실제 실험을 근거로 발전시킵니다. 관찰과 변경 내용은
[`experiments/`](experiments/)에서 확인할 수 있습니다. 이는 근거 기록이며
통계적으로 유의한 벤치마크 주장은 아닙니다.

## 라이선스

MIT
