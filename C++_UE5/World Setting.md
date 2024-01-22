
## 엔진 기본 구조
![[UnrealWorld.png]]
- World: 복수의 레벨을 포함할 수 있으며 각각의 레벨을 스트리밍하고 액터를 스폰하는역할
- GameInstance: 게임 인스턴스는 게임 내에 단 하나의 객체만 생성되며 게임이 실행될 때 생성이 되고 게임이 종료되면 파괴된다. 언리얼 엔진에서는 전역 변수를 사용하지 않기 때문에 필요한 경우 게임 인스턴스 내에 정의하여 사용한다.
- Level: 게임에서 사용되는 맵. 레벨에는 갖가지 액터들이 배치된다.
- GameMode: 레벨 당 한 개의 게임 모드를 가지며 게임의 룰을 정의한다.
- GameStates: 게임의 상태 정의. 게임이 시작되었는지 중단되었는지 팀의 스코어가 몇점인지 등을 포함한다
- PlayerStates: 플레이어의 상태 정의. 플레이어의 캐릭터가 무슨 직업인지 이름은 무엇인지 등을 포함한다
- Actors: 레벨에 배치할 수 있는 모든 객체를 의미. 실제의 게임 화면에 보여지든 아니든 레벨에 배치된다면 해당 객체는 액터 클래스를 반드시 상속 받아야 한다. 복수의 컴포넌트를 포함할 수 있다
- Objects: 레벨에 배치되지 않는 객체로 기능으로서만 존재. 캐릭터의 인벤토리 시스템이나 스킬 시스템 등을 구현하기 알맞다
- Components: 컴포넌트는 액터에 반드시 포함되어야 하며 각각의 컴포넌트들은 특징적인 기능을 갖는다. 스태틱 메쉬나 스켈라탈 메쉬를 화면에 출력하거나 파티클을 출력하는 등 수많은 컴포넌트가 존재

## Class 네이밍
- **Actor** (액터) 파생 클래스 접두사는 **A** 입니다. 예: AController.
- **Object** (오브젝트) 파생 클래스 접두사는 **U** 입니다. 예: UComponent.
- **Enum** (열거형) 접두사는 **E** 입니다. 예: EFortificationType.
- **Interface** (인터페이스) 클래스 접두사는 보통 **I** 입니다. 예: IAbilitySystemInterface.
- **Template** (템플릿) 클래스 접두사는 **T** 입니다. 예: TArray.
- **SWidget** (슬레이트 UI) 파생 클래스 접두사는 **S** 입니다. 예: SButton.
- 그 외의 접두사는 **F** 입니다. 예: FVector.



## 작업 순서
1. Login(플레이어 입장) 중에 게임 모드에 의해 Player Controller 생성
2. Game Mode에 의해 Player Pawn 생성
3. Player Controller에 의해 Player Controller가 지정된 Pawn에 빙의하려 함

### 게임모드
```
#include "CoreMinimal.h"
#include "GameFramework/GameModeBase.h"
#include "IngameGameMode.generated.h"

UCLASS(minimalapi)
class AIngameGameMode : public AGameModeBase
{
	GENERATED_BODY()

public:
	AIngameGameMode();	
};
```

```
#include "IngameGameMode.h"
#include "IngameHUD.h"
#include "IngamePlayerController.h"

AIngameGameMode::AIngameGameMode()
{
	// set default pawn class to our Blueprinted character
	static ConstructorHelpers::FClassFinder<APawn> PlayerPawnBPClass(TEXT("/Game/RPG/Character/BP_IngameCharacter"));
	
	if (PlayerPawnBPClass.Class != NULL)
	{
		DefaultPawnClass = PlayerPawnBPClass.Class;
	}
	
	HUDClass = AIngameHUD::StaticClass();     // HUD 클래스
	PlayerControllerClass = AIngamePlayerController::StaticClass(); // 플레이어 컨트롤러 클래스
}
```


### HUD 클래스
```
#include "CoreMinimal.h"
#include "GameFramework/HUD.h"
#include "IngameHUD.generated.h"

UCLASS()
class ROLEPLAYINGGAMECODE_API AIngameHUD : public AHUD
{
	GENERATED_BODY()

protected:
	// Called when the game starts or when spawned
	virtual void BeginPlay() override;	
};
```

```
#include "IngameHUD.h"

void AIngameHUD::BeginPlay()
{
	Super::BeginPlay();
	
	APlayerController* pPC = Cast<APlayerController>(GetWorld()->GetFirstPlayerController());
	pPC->SetInputMode(FInputModeGameAndUI());     // 게임 모드 설정
	pPC->SetShowMouseCursor(true);

}
```



### 디폰트 폰 클래스
```
#include "CoreMinimal.h"
#include "GameFramework/Character.h"
#include "InputActionValue.h"
#include "IngameCharacter.generated.h"

UCLASS()
class ROLEPLAYINGGAMECODE_API AIngameCharacter : public ACharacter, public IDamageableInterface
{
	GENERATED_BODY()

	/** Camera boom positioning the camera behind the character */
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Camera, meta = (AllowPrivateAccess = "true"))
	class USpringArmComponent* CameraBoom;

	/** Follow camera */
	UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = Camera, meta = (AllowPrivateAccess = "true"))
	class UCameraComponent* FollowCamera;

	/** MappingContext */
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input, meta = (AllowPrivateAccess = "true"))
	class UInputMappingContext* DefaultMappingContext;

	/** Jump Input Action */
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input, meta = (AllowPrivateAccess = "true"))
	class UInputAction* JumpAction;

	/** Move Input Action */
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input, meta = (AllowPrivateAccess = "true"))
	class UInputAction* MoveAction;

	/** Look Input Action */
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input, meta = (AllowPrivateAccess = "true"))
	class UInputAction* LookAction;

	/** InteractionE Input Action */
	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input, meta = (AllowPrivateAccess = "true"))
	class UInputAction* InteractionAction;

private:
	UPROPERTY(EditAnywhere, Category = Weapon, meta = (AllowPrivateAccess = "true"))
	TSubclassOf<class AWeapon> Weapon;

	UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = Input, meta = (AllowPrivateAccess = "true"))
	class UDamageSystemActorComp* DamageSystem;

public:
	// Sets default values for this character's properties
	AIngameCharacter();

protected:

	/** Called for movement input */
	void Move(const FInputActionValue& Value);

	/** Called for looking input */
	void Look(const FInputActionValue& Value);

	/** Called for looking input */
	void Interaction(const FInputActionValue& Value);

protected:
	virtual void BeginPlay() override;

public:	
	/** Returns CameraBoom subobject **/
	FORCEINLINE class USpringArmComponent* GetCameraBoom() const { return CameraBoom; }
	/** Returns FollowCamera subobject **/
	FORCEINLINE class UCameraComponent* GetFollowCamera() const { return FollowCamera; }
};
```


## Enhanced Input
- #EnhancedInput 은 UE5에서 런타임 리매핑, 복잡한 입력 처리(동시 입력) 등 **향상된 입력 기능**을 제공하는 플러그인 (5.1버전 이상이면 기본적으로 플러그인이 설치되어 있음)
- <mark class="hltr-orange">Input Action</mark>: 액션이 할당되는 부분, 특정 키가 연결되지 않고 역할에 대한 정보 만을 구성
	- 프로그램에서 IA_Jump, IA_Look, IA_Move 와 같은 얘
	- 입력 받는 정보(Value Type)는 bool, float, Vector2D, Vector3D 를 받을 수 있다
	- '문을 연다', '장비를 착용한다' 와 같은 동작은 bool 값으로 지정하고 이동 같은 동작은 Vector2D 로 설정
- <mark class="hltr-orange">Input Mapping Context</mark>: 사용자의 입력 값을 만들어둔 input action과 바인딩.
	- 사용자는 여러 개의 Input Mapping Context 를 가질 수 있으며 이들은 각각 우선순위가 있어 "같은 키를 입력해도 어떤 액션이 나갈 지를 지정할 수 있다!!"
- Modifier : 입력 받은 값을 반환해주는 장치
	- Swizzle Input Axis Values: 원래 입력을 받으면 XYZ 순서로 받게 되는 데 YXZ의 순서로 받게 하여 Y가 1이 되도록 설정할 수 있다
	- Negate: 입력 받은 값을 반대로 해주는 역할
- Trigger: Modifier를 통해 입력 받은 값을 어떻게 활용할지 정하게 됨
- (Tip) UInputAction 은 # include를 하게 되면 컴파일 시간이 증가하므로 `Forward Declaration`(전방 선언) 해주기 -> C++에서는 하나의 헤더파일이 변경되면 이를  include 하는 모든 파일이 다시 컴파일 되기 때문에 이를 방지하기 위해 class라고 명시