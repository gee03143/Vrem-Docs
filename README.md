# Vrem-Docs

Vrem 프로젝트의 공개용 문서 레포입니다. 사양 문서와 원본 프로젝트에서 캡처된 gif, png 파일들로 구성되어 있습니다

문서가 인용하는 C++ 코드는 `Source/`에 함께 두었습니다. 원본 프로젝트의 **`9d396a3` (2026-10-07)** 시점 스냅샷이며, 에셋은 라이선스상 포함하지 않아 이 저장소만으로는 빌드되지 않습니다.

## Vrem

Unreal Engine 5.7 기반 3인칭 슈팅 액션 프로토타입.

멀티플레이를 전제로 인벤토리 · 장비 · 무기 · 전투 시스템을 C++ 프레임워크로 설계하고,

게임플레이 조립과 정책은 블루프린트에 맡기는 하이브리드 구조로 만들었습니다.

![무기 스왑과 사격](docs/media/swap-shooting.gif)

## 설계에서 중요하게 본 것

**개발 과정에서 게임 디자이너와의 협력을 전제로 개발하였습니다**

네이티브 C++에서 핵심 프레임워크를 작성하고, 블루프린트로 세부적인 스펙을 정의합니다

디자이너가 수정사항을 요청하고 DLL 빌드를 기다리는 대신, 자유롭게 테스트할 수 있도록 프레임워크를 구축합니다

![WD_Rifle 데이터 에셋](docs/media/WeaponDefinition.png)

캐릭터와 아이템을 **컴포넌트와 데이터 에셋의 조합으로 조립**하는 구조

![BP_DefaultCharacter 컴포넌트 구성](docs/media/CharacterBlueprint-Components.png)

![ID_HeavyAmmo 데이터 에셋](docs/media/ItemDefinition.png)

**판정과 상태는 서버가 쥐고, AI도 같은 경로를 씁니다**

히트스캔, 근접 히트 판정, 회피 힘 적용은 전부 서버에서만 실행됩니다. 클라이언트는 입력에 즉시 반응하되 서버가 복제한 값으로 보정받습니다.

AI는 플레이어와 같은 컴포넌트를 쓰고 같은 함수로 들어옵니다. 그래서 인벤토리에서 탄을 꺼내 쓰고, 재장전하고, 서버 검증을 똑같이 통과합니다. AI를 위해 따로 구현한 사격이 없으니 한쪽만 고쳐져 어긋날 일도 없습니다.

![AI 전투](docs/media/ai-usesamecomponent-attack.gif)

두 클라이언트를 나란히 띄운 화면이고 주황색이 AI입니다. 같은 전투가 양쪽에 그려지고 체력 변화도 함께 반영되지만, 오가는 것은 애니메이션이 아니라 콤보 인덱스와 상태값입니다.

무엇을 누구에게 보낼지도 나눴습니다. 같은 FastArray여도 복제 조건이 갈립니다.

```cpp
// 가방 속 내용은 본인만 보면 된다
DOREPLIFETIME_CONDITION(UVremInventoryComponent, InventoryItems, COND_OwnerOnly);

// 손에 든 무기는 모두에게 그려져야 한다
DOREPLIFETIME(UVremEquipmentComponent, EquipmentList);
```

사격 경로에는 서버가 클라이언트 요청을 다시 판단하는 검증을 추가했습니다.

## 시스템별 상세 문서

| 문서 | 다루는 것 |
|---|---|
| **[레이어 분리와 복제 규약](docs/architecture/layering.md)** | C++와 블루프린트의 경계를 어디에 왜 그었나. 이미 작성한 C++를 되돌린 판단. 무엇을 누구에게 복제할지의 규칙 |
| [아이템 파이프라인](docs/architecture/items.md) | `Definition` + `Fragment` 합성으로 아이템을 정의하고, 인벤토리 → 장비 → 무기로 흘려보내는 구조 |
| [전투와 애니메이션](docs/architecture/combat.md) | 사격 1발이 클라이언트와 서버를 오가는 과정. 근접 콤보 상태머신. 무기별 애니메이션 교체 |
| [자동화 테스트](docs/architecture/testing.md) | 자동화 테스팅 구축을 통한 회귀 버그 관리 |
