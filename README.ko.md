# Red Green Prune

**테스트 비대화와 과잉 설계 없는 TDD.**

[English](README.md)

코딩 에이전트는 종종 구현부터 시작하고 나중에 테스트를 추가하면서도
TDD를 했다고 말합니다. 경계 조건을 필요 이상으로 세분화하고, 같은 의미의
테스트를 반복하며, 필요가 입증되기 전에 추상화를 도입하고, 주변 코드에
있다는 이유만으로 어노테이션을 복사합니다.

Red Green Prune은 에이전트에 더 엄격하고 작은 반복 과정을 적용합니다.

```text
ALIGN -> RED -> GREEN -> REFACTOR -> PRUNE
```

동작을 이해합니다. 실제 실패를 관찰합니다. 통과하는 가장 작은 변경을
작성합니다. 현재 변경이 필요성을 증명한 부분만 개선합니다. 새로운 테스트
비대화를 막습니다.

## 적용 전 / 적용 후

사용자 이름을 대소문자 구분 없이 조회하도록 요청했다고 가정해 봅시다.

Red Green Prune이 없다면 에이전트는 곧바로 기능을 구현하고, 그 뒤 여러
테스트를 추가하며, `UsernameNormalizer` 추상화를 도입하고, 일관성을 이유로
인접 클래스의 어노테이션을 복사할 수 있습니다.

Red Green Prune을 사용하면 다음 순서로 작업합니다.

```text
ALIGN     조회 경로와 기존 테스트를 읽고 저장소에서 동작 의미를 확인합니다.
RED       가장 가까운 테스트를 확장하고 예상한 실패를 관찰합니다.
GREEN     기존 비교 로직을 대소문자 구분 없이 동작하도록 변경합니다.
REFACTOR  필요성이 입증된 문제가 없으므로 아무것도 변경하지 않습니다.
PRUNE     RED를 증명한 테스트만 남기고 같은 의미의 사례는 추가하지 않습니다.
```

결과는 기존 구현에 더 큰 절차만 덧씌우는 것이 아니라, 더 적은 프로덕션
코드와 더 적은 테스트로 이루어진 실제 테스트 우선 개발입니다.

## 모듈형 스킬

현재 작업에 필요한 정책만 로드하도록 책임별로 스킬을 분리했습니다.

| 스킬 | 책임 | 사용 대상 |
| --- | --- | --- |
| `red-green-prune` | 얇은 진입점과 라우팅 | 전체 작업 과정 |
| `test-first-cycle` | ALIGN, 정직한 RED 근거, 규칙별 사이클 | 기능, 수정, API, 계산, 검증 |
| `minimal-change` | 최소 GREEN과 절제된 REFACTOR | 구현, 과잉 설계, 불필요한 어노테이션 |
| `test-prune` | 중복 테스트 근거와 삭제 안전성 | 겹칠 수 있는 새 테스트 또는 명시적 테스트 정리 |

일반적인 동작 변경에는 `test-first-cycle`과 `minimal-change`를 사용합니다.
`test-prune`은 테스트가 겹칠 가능성이 있거나 정리를 요청받았을 때만
로드합니다. 모든 규칙을 함께 읽는 references로 옮긴 것이 아니라 독립
스킬로 분리했으므로 평소 컨텍스트 비용을 줄일 수 있습니다.

## 작동 방식

### ALIGN과 RED

수정하기 전에 영향받는 코드, 호출자, 가장 가까운 테스트를 읽습니다.
저장소 사실을 직접 확인하고, 독립적으로 실패할 수 있는 규칙 하나를 고른
뒤 프로덕션 코드 변경 전에 실제 실패를 관찰합니다. 통과 테스트, guard,
놓친 RED를 TDD 근거로 바꿔 부르지 않고 정직하게 보고합니다.

### GREEN과 REFACTOR

선택한 규칙과 기존 계약을 만족하는 가장 작은 프로덕션 변경을 작성합니다.
추측에 기반한 추상화, 설정, 의존성, fallback, 어노테이션, 관련 없는 정리를
추가하지 않습니다. 테스트가 통과하고 현재 변경이 실제 설계 문제를
증명했을 때만 리팩터링합니다.

### PRUNE

현재 작업에서 생긴 중복 테스트를 방지합니다. 서로 다른 동작, 경계,
동등 클래스, 회귀, 보안 규칙, 실패 형태를 보호하는 테스트는 유지합니다.

## 안전하게 줄이기

Red Green Prune은 새로운 테스트 비대화를 방지합니다. 기존 테스트 모음을
자율적으로 정리하지 않습니다.

일반적인 기능 구현이나 버그 수정 중에는 기존 테스트를 삭제하거나,
비활성화하거나, 약화해서는 안 됩니다. 기존 테스트 중 중복으로 의심되는
항목은 별도로 보고합니다. 이를 제거하려면 명시적인 정리 요청이 필요하며,
위험도가 높은 테스트는 근거와 사용자 승인도 필요합니다.

이 제한은 의도적입니다. 기존 테스트 하나가 더 있으면 유지보수 시간이
늘어납니다. 하지만 회귀 테스트를 잘못 삭제하면 프로덕션 사고로 이어질 수
있습니다.

## 설치

### Codex

```bash
codex plugin marketplace add irerin07/red-green-prune
codex plugin add red-green-prune@red-green-prune
```

새 Codex 세션을 시작한 뒤 전체 작업 과정을 명시적으로 호출합니다.

```text
$red-green-prune 대소문자를 구분하지 않는 사용자 이름 조회를 구현해줘
```

Codex는 작업 내용에 따라 개별 스킬을 자동으로 활성화할 수도 있습니다.

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

플러그인을 업데이트했다면 Claude Code가 최신 스킬 파일을 발견하도록 새
세션을 시작하세요.

## 하지 않는 것

- 문서, 포매팅, 생성된 파일, 동작을 바꾸지 않는 이름 변경, 사용하지 않는
  코드 삭제에는 RED를 요구하지 않습니다.
- 기존 저장소에는 요청 없이 테스트 프레임워크를 만들지 않습니다. 새
  프로젝트에는 가장 작은 표준 테스트 환경을 구성합니다.
- 일반적인 구현 작업 중 기존 테스트 커버리지를 삭제하지 않습니다.
- 요구사항을 충족한 뒤 선택적인 개선 사항까지 구현하지 않습니다.
- RED 단계를 관찰하지 않았다면 TDD를 했다고 주장하지 않습니다.

스킬 정의는 다음 파일에 있습니다.

- [`skills/red-green-prune/SKILL.md`](skills/red-green-prune/SKILL.md)
- [`skills/test-first-cycle/SKILL.md`](skills/test-first-cycle/SKILL.md)
- [`skills/minimal-change/SKILL.md`](skills/minimal-change/SKILL.md)
- [`skills/test-prune/SKILL.md`](skills/test-prune/SKILL.md)

## 현재 상태

규칙은 기록된 실제 실험을 근거로 발전시킵니다. 관찰 내용과 그로 인한
변경은 [`experiments/`](experiments/)에서 확인할 수 있습니다. 이는 근거
기록이며 통계적으로 유의한 벤치마크 주장은 아닙니다.

LOC, 토큰, 비용, 시간, 안전성 벤치마크를 준비하고 있습니다. 아직 성능
개선 수치를 주장하지 않습니다.

## 라이선스

MIT
