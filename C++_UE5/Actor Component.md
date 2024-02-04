## 액터 컴포넌트
- 액터는 월드(레벨)에 배치될 수 있는 최소 단위
- 컴포넌트는 액터가 자기 자신에 서브 오브젝트로 어태치할 수 있는 특수한 타입의 오브젝트
- `UActorComponent`는 모든 컴포넌트에 대한 베이스 클래스
- 계층 구조를 가지는 컴포넌트는 Scene Component, 기능만 가지는 컴포넌트는 Actor Component
![[액터 컴포넌트.png]]

### 캐릭터 컴포넌트에 액터 컴포넌트 추가
```
class ROLEPLAYINGGAMECODE_API AIngameCharacter : public ACharacter
{
public:
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input, meta = (AllowPrivateAccess = "true"))
class UDamageSystemActorComp* DamageSystem;
}
```

```
AIngameCharacter::AIngameCharacter()
{
	DamageSystem = CreateDefaultSubobject< UDamageSystemActorComp>(TEXT("DamageSystem"));
this->AddOwnedComponent(DamageSystem);
}

```