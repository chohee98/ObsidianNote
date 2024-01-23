### Build.cs
```
using UnrealBuildTool;

public class RolePlayingGameCode : ModuleRules
{
	public RolePlayingGameCode(ReadOnlyTargetRules Target) : base(Target)
	{
		PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;

		PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "HeadMountedDisplay", "EnhancedInput", "Niagara", "UMG" });

        PrivateDependencyModuleNames.AddRange(new string[] { });
    }
}
```

### Engine
- C++ "**UserWidget**" 클래스 생성
- 블루프린트로 "위젯 블루프린트" 생성