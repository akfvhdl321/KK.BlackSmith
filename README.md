<!-- ============================================================
  깡깡대장간 (Kkangkkang Blacksmith) — 개인 포트폴리오용 README
  ▸ "⚠️" 로 표시된 곳은 실제 값으로 채운 뒤 표시를 삭제하세요.
  ▸ 🔑 주요 코드 파일 표의 링크는 본인 레포 주소로 교체하세요.
  ▸ 🚨 코드 관련 문장(클래스/메서드명, 동작 설명)은 게시 전
     실제 소스 파일과 반드시 대조 검증하세요. (설계 문서 ≠ 실제 코드)
============================================================ -->

<p align="center">
  <img alt="게임 대표 이미지" src="./Assets/Docs/Imgs/readmeTitle.png" width="640" />
</p>

## 깡깡대장간 : Kkangkkang Blacksmith ⚒️  <!-- ⚠️ 영문 표기명 확인 -->

> **바탕화면 위에 얹히는 클릭-스루(click-through) 위젯형 방치 RPG.**
> 대장간에서 장비를 제작하고, 파티를 모험에 내보내 재화를 모으는 순환형 게임.

---

### 🗂️ 프로젝트 개요

- **프로젝트명** : 깡깡대장간 ( Kkangkkang Blacksmith )
- **장르** : 바탕화면 위젯형 방치 RPG ( Desktop Widget · Idle )
- **플랫폼** : PC ( Windows )
- **엔진 / 언어** : Unity 6 (2D) / C#
- **개발 기간** : 약 3주  <!-- ⚠️ 정확한 날짜로 교체: 예) 2026.03.xx ~ 2026.03.xx -->
- **인원** : 3명  <!-- ⚠️ 표기 방식 확인: 예) 초기 5명 → 개발 중 3명 -->
- **역할** : 모험 · 퀘스트 · 사운드 · 현지화 · 환경설정 · 던전 배경 담당

---

### 🧩 기술 스택

| 분류 | 사용 기술 |
| --- | --- |
| 엔진 · 언어 | Unity 6 (2D), C# |
| 창 제어 | UniWindowController (Win32 `SetWindowLong` / `WS_EX_TRANSPARENT` — 클릭 스루) |
| 데이터 | CSV (던전 · 현지화 데이터), Excel 데이터 테이블 |
| 버전 관리 | Git · GitHub |
| 설계 · 패턴 | 상태 머신(FSM), 시간 기반 진행률 이동, 매니저(Manager) 구조 |

---

### 🎮 게임 소개

#### 🖥️ 바탕화면 위젯 (클릭 스루)

- 게임 화면이 **바탕화면 위에 투명하게 얹혀** 작동합니다. 대장간 건물·파티 등 상호작용 요소를 뺀 빈 영역은 **클릭이 그대로 바탕화면으로 통과**됩니다.
- `UniWindowController`로 Win32 API(`SetWindowLong` + `WS_EX_TRANSPARENT`)를 제어해, 픽셀 단위로 클릭 통과/차단을 전환합니다.

#### ⚒️ 대장간 & 순환 루프

- 재화로 **장비를 제작**하고, 제작한 장비로 파티를 강화합니다.
- 강화한 파티를 **모험(퀘스트)** 에 내보내 재화를 획득 → 다시 제작에 투자하는 **방치형 순환 구조**입니다.

#### 🧭 모험(퀘스트) 연출

- 파티가 직접 조작되는 것이 아니라, **정해진 퀘스트 시간에 맞춰 월드맵을 연출적으로 이동**합니다.
- 이동 중 **발견 · 야영 · 던전 전투** 이벤트가 위치에 따라 자동 발동됩니다.

---

### 🕹️ 조작 방법

<!-- ⚠️ 실제 기본 조작/단축키로 교체하세요. 환경설정에서 단축키 변경 가능 -->

| 대상 | 조작 | 동작 |
| --- | --- | --- |
| 대장간 건물(최소화) | 마우스 좌클릭 | 대장간 메인 화면 열기 |
| 컨트롤 UI | 마우스 좌클릭 | 퀘스트 보드 · 환경설정 등 팝업 열기 |
| 환경설정 | 사용자 지정 단축키 | 팝업 토글 (⚠️ 기본키 확인) |

---

### 👤 담당 파트 — 모험 · 퀘스트 · 사운드 · 현지화 · 환경설정 · 던전 배경

> 3인 팀 프로젝트에서 아래 시스템을 직접 설계·구현했습니다.
> (화면 전환 · 보상/자루 팝업 · `PopupManager` · 최소화 월드맵은 팀장,
>  대장간/장비 제작 시스템은 다른 팀원이 담당했습니다.)

#### 🧭 모험 시스템  `AdventureManager`

- **상태 머신(FSM)** 으로 모험 전 과정을 관리합니다.
  `None → Going → (Discovering / Camping / Battle) → Celebrate → Returning → Completed`
- 이동은 **시간 기반 진행률**로 처리합니다. `normalized = elapsedTime / questDuration` 을 기준으로 위치를 `Lerp` 계산해, **속도 값을 저장하지 않는 무상태(stateless) 이동**으로 설계 → 앱을 재실행해도 위치를 그대로 복원할 수 있게 했습니다.
- **`SyncFromQuestTime`** 으로 앱 종료 시각과 재실행 시각의 **`DateTime` 차이**를 계산해, 방치 중 진행됐어야 할 위치·상태를 복원합니다.
- 발견/야영/전투는 **위치 기반 이벤트 존**으로 판정하고, 비주얼은 Animator 없이 **오브젝트 활성/비활성 토글**로 처리해 로직과 분리했습니다.

#### 📜 퀘스트 시스템  `QuestManager` · 퀘스트 보드 UI

- 퀘스트 진행/완료 상태를 관리하고, **완료 판정을 단일 타이밍 지점으로 통일**해 `DateTime` 기반 시각과 `Time.deltaTime` 누적 사이의 오차(desync)를 방지했습니다.
- 완료된 퀘스트는 `_completedQuest`에 별도 보관하고, **미수령 보상이 있으면 새 출발(`StartQuest`)을 차단**하도록 설계했습니다.
- 완료 여부는 프로퍼티(`HasCompletedQuest`) 폴링으로 노출하고, 수령 시 `ClearCompletedQuest()`로 정리합니다.

#### 🔊 사운드 매니저  `SoundManager`

- BGM / SFX 재생·볼륨 제어를 중앙에서 담당하는 사운드 허브를 구현했습니다.  <!-- ⚠️ 실제 기능 범위로 보완 -->

#### ⚙️ 환경설정 팝업

- **단축키 커스터마이징** 기능을 구현하고, **중복 키 입력을 감지·처리**해 같은 키가 여러 기능에 배정되는 충돌을 방지했습니다.

#### 🏔️ 던전 배경 시스템

- 던전 데이터(CSV)에 따라 배경을 전환하는 시스템을 구현했습니다.  <!-- ⚠️ 실제 동작으로 보완 -->

#### 🎞️ 파티 애니메이션

- 기본 / 전투 / 야영(텐트) / 자축 등 상태별 파티 연출과 말풍선(`?` `!` `zzz`) 토글을 구현했습니다.

---

### 🖼️ 구현 화면

<table>
  <tr>
    <td align="center">
      <img alt="모험 화면" src="https://github.com/user-attachments/assets/df50e66c-ab41-437b-9a26-16640922e257" width="400" /><br/>
      모험 화면 (발견 · 야영 · 던전)
    </td>
    <td align="center">
      <img alt="퀘스트 보드" src="https://github.com/user-attachments/assets/0e301f5f-919e-44d8-b111-e64350d8a6e1" width="400" /><br/>
      퀘스트 보드 팝업
    </td>
  </tr>
  <tr>
    <td align="center">
      <img alt="환경설정" src="https://github.com/user-attachments/assets/58b75370-5ee6-4f56-8c08-efa4220a7ed2" width="400" /><br/>
      환경설정 (단축키 커스터마이징)
    </td>
    <td align="center">
      <img alt="최소화 화면" src="https://github.com/user-attachments/assets/40b35dd9-eec5-4dd2-9b9f-c0ba6f1f0b54" width="400" /><br/>
      최소화 위젯 화면
    </td>
  </tr>
</table>

---

### 📂 프로젝트 구조

<!-- ⚠️ 실제 레포 폴더 구조에 맞춰 보완하세요. -->

```
Assets/
├─ Docs/            → 기획 · 설계 문서
├─ Scripts/
│  ├─ Adventure/    → AdventureManager (FSM · 시간 기반 이동 · 상태 복원)   ← 담당
│  ├─ Quest/        → QuestManager · 퀘스트 보드 UI                        ← 담당
│  ├─ Sound/        → SoundManager                                         ← 담당
│  ├─ Localization/ → LocalizationManager (CSV)                            ← 담당
│  ├─ Settings/     → 환경설정 팝업 (단축키 커스터마이징)                   ← 담당
│  ├─ Dungeon/      → 던전 배경 시스템                                      ← 담당
│  ├─ Popup/        → PopupManager · 화면 전환 · 보상/자루 팝업            (팀장)
│  └─ Smithy/       → 대장간 · 장비 제작 시스템                            (팀원)
├─ Resources/       → CSV (던전 · 현지화), 아이콘 리소스
└─ Plugins/         → UniWindowController
```

---

### 🏛️ 핵심 설계 — 모험 시스템 "상태 복원" 체인

> 방치형 게임의 핵심 난제인 **"앱을 껐다 켜도 진행 상황이 이어져야 한다"** 를
> 아래 네 단계의 연결된 설계로 풀었습니다.

| 단계 | 설계 | 해결한 문제 |
| --- | --- | --- |
| ① 상태 머신 (FSM) | `enum` 상태 + 전이 규칙으로 모험 흐름을 명시적으로 관리 | 이벤트/연출 분기의 복잡도 제어 |
| ② 시간 기반 이동 | 속도 대신 **진행률(`elapsedTime / questDuration`)** 로 위치 계산 | 이동을 **무상태(stateless)** 로 만들어 복원 가능하게 함 |
| ③ `SyncFromQuestTime` | 종료↔재실행 **`DateTime` 차이**로 위치·상태 복원 | 방치 중 진행분 반영 |
| ④ 완료 타이밍 통일 | 완료 판정을 단일 지점으로 통일 | `DateTime` vs `Time.deltaTime` 누적 오차(desync) 제거 |

> 💡 **회고** : 현재는 `enum` + `switch` 구조라 상태별 독립 클래스가 없습니다.
> 상태가 더 늘어날 경우 **State 패턴(상태별 클래스 분리)** 이 확장성에 유리했을 것으로 판단합니다.

---

### 🔑 주요 코드 파일 <sub>(담당)</sub>

<!-- ⚠️ 링크의 {레포주소}/{브랜치}/{경로} 를 실제 GitHub 주소로 교체하세요. -->
<!-- ⚠️ 클래스/파일명은 실제 소스와 대조 후 확정하세요. -->

| 파일 | 설명 | 링크 |
| --- | --- | --- |
| `AdventureManager.cs` | 모험 FSM. 시간 기반 진행률 이동, `SyncFromQuestTime` 상태 복원, 위치 기반 이벤트 판정 | [📄](https://github.com/akfvhd1321/KK.BlackSmith/blob/develop/Assets/CWT/CWT_Scripts/Manager/AdventureManager.cs) |
| `QuestManager.cs` | 퀘스트 완료 타이밍 통일, `_completedQuest` 보관, `HasCompletedQuest` 폴링, 미수령 시 `StartQuest` 차단 | [📄](https://github.com/akfvhd1321/KK.BlackSmith/blob/develop/Assets/CWT/CWT_Scripts/Manager/QuestManager.cs) |
| `SoundManager.cs` | BGM/SFX 재생·볼륨 중앙 관리 | [📄](https://github.com/akfvhd1321/KK.BlackSmith/blob/develop/Assets/CWT/CWT_Scripts/Manager/SoundManager.cs) |
---

### 🎬 화면 구성

<!-- ⚠️ 실제 씬/화면 구성으로 보완하세요. -->

| 화면 | 역할 |
| --- | --- |
| 최소화 위젯 | 바탕화면 상주 · 대장간 건물 클릭 진입 |
| 대장간 메인 | 장비 제작 · 파티 관리 |
| 모험(퀘스트) | 시간 기반 파티 이동 · 이벤트 연출 |
| 각종 팝업 | 퀘스트 보드 · 환경설정 · 보상/자루 등 |

---
