## 인터페이스
- 특정 기능을 구현할 것을 약속한 추상 형식
- (잠재적으로) 무관한 클래스 세트가 공통의 함수 세트를 구현할 수 있도록 하는데 쓰임
- 유사성이 없었을 크고 복잡한 클래스들에 어떤 게임 함수 기능을 공유 시키고자 하는 경우 인터페이스 사용
-  예를 들어 트리거 볼륨에 들어서면 함정이 발동되거나, 적에게 경보가 울리거나, 플레이어에게 점수를 주는 시스템을 가진 게임이 있다 칩시다. 함정, 적, 점수에서 ReactToTrigger 함수를 구현해 각각 위의 동작을 구현하면 될 것입니다. 하지만 함정은 AActor에서 파생될 수도, 적은 특수 APawn이나 ACharacter의 서브클래스일 수도, 점수는 UDateAsset일 수도 있습니다. 이 모든 클래스에 공유 함수 기능이 필요하지만, UObejct 말고는 공통 조상이 없습니다. 이럴 때 인터페이스를 쓰는 것을 추천합니다. 



### 캐릭터에 데미지 인터페이스 적용
```
#include "CoreMinimal.h"
#include "UObject/Interface.h"
#include "DamageableInterface.generated.h"

// This class does not need to be modified.
UINTERFACE(MinimalAPI)
class UDamageableInterface : public UInterface
{
	GENERATED_BODY()
};

class ROLEPLAYINGGAMECODE_API IDamageableInterface
{
	GENERATED_BODY()

	// Add interface functions to this class. This is the class that will be inherited to implement this interface.
public:
	UFUNCTION()
	virtual float CurHp();

	UFUNCTION()
	virtual float MaxHp();

	UFUNCTION()
	virtual float GetHeal(float HealAmount);

	UFUNCTION()
	virtual float TakeDamage(float DamageAmount);
};
```

```
float IDamageableInterface::CurHp()
{
	return 0.0f;
}

float IDamageableInterface::MaxHp()
{
	return 0.0f;
}

float IDamageableInterface::GetHeal(float HealAmount)
{
	return 0.0f;
}

float IDamageableInterface::TakeDamage(float DamageAmount)
{
	return 0.0f;
}
```

```
#include "DamageableInterface.h"
UCLASS()
class ROLEPLAYINGGAMECODE_API AIngameCharacter : public ACharacter, public IDamageableInterface
{
public:
	virtual float CurHp() override;
	virtual float MaxHp() override;
	virtual float GetHeal(float HealAmount) override;
	virtual float TakeDamage(float DamageAmount) override;
}
```

```
void AIngameCharacter::Interaction(const FInputActionValue& Value)  // 임시로 인터페이스 확인용!! 
{
	IDamageableInterface* Interface = Cast<IDamageableInterface>(GetCapsuleComponent()->GetOwner());
	if (Interface != nullptr)
	{
		Interface->TakeDamage(1000);
	}
}
float AIngameCharacter::CurHp()
{
	return DamageSystem->CurHp;
}

float AIngameCharacter::MaxHp()
{
	return 0.0f;
}
float AIngameCharacter::GetHeal(float HealAmount)
{
	return 0.0f;
}
float AIngameCharacter::TakeDamage(float DamageAmount)
{
	
	return DamageSystem->TakeDamage(DamageAmount);
}
```


