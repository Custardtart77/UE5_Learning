
# C++ UActor & Uclass编程笔记
## 创建Component对象，如UStaticMeshComponent，返回指针类型

UStaticMeshComponent* CubeMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("CubeMesh"));
UBoxComponent* TriggerBox = CreateDefaultSubobject<UBoxComponent>(TEXT("TriggerBox"));

对应CubeMesh，实际还没有真正的Static mesh，需要UPROPERTY声明后在detail面板中传入

或者直接在代码里面读取

### 代码里面读取
    // 使用ConstructorHelpers::FObjectFinder读取Obect对象，再将CubeVisualAsset.Object赋予SetStaticMesh
    static ConstructorHelpers::FObjectFinder<UStaticMesh> CubeVisualAsset(TEXT("/Game/StarterContent/Shapes/Shape_Cube.Shape_Cube"));

    if (CubeVisualAsset.Succeeded())
    {
        VisualMesh->SetStaticMesh(CubeVisualAsset.Object);
        VisualMesh->SetRelativeLocation(FVector(0.0f, 0.0f, 0.0f));
    }

    // 同理，网格体的材质也可以这样设置
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "UI")
	UMaterialInstance* CorrectCubeMaterial;


    static ConstructorHelpers::FObjectFinder<UMaterialInstance> CubeVisualAsset(TEXT("/Game/StarterContent/Shapes/));
    if (CubeVisualAsset.Succeeded())
    {
        CorrectCubeMaterial = CubeVisualAsset.Object;
    }
    
    CubeMesh->SetMaterial(0, CorrectCubeMaterial);

## 创建材质、UI控件类型等


 ### 最简便的方法就是声明相关对象指针，然后在Detail面板中传入
	// 显示的 UI 按钮，不加UPROPERTY可能会被异常回收
	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "UI")
	UUserWidget* InteractionButtonWidget;

	UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "UI")
	UMaterialInstance* CorrectCubeMaterial;

    // 使用的时候要注意判断
    if (InteractionButtonWidget)
    {
        // ...

    }

### 代码里面读入

方法1：先加载类，再根据类创建对象； 类的加载可以detail面板加载，也可以ConstructorHelpers::FClassFinder加载

    // 使用 ConstructorHelpers 加载蓝图类
    static ConstructorHelpers::FClassFinder<UUserWidget> InteractionButtonWidgetBP(TEXT("/Game/UI/BP_InteractionButton")); // 蓝图的路径

    // 确保成功加载蓝图
    if (InteractionButtonWidgetBP.Succeeded())
    {
        InteractionButtonWidgetClass = InteractionButtonWidgetBP.Class;
    }
    else
    {
        UE_LOG(LogTemp, Warning, TEXT("Failed to find BP_InteractionButton!"));
    }
    */

    // 确保我们有正确的 Widget 蓝图
    if (InteractionButtonWidgetClass)
    {
        // 创建按钮 Widget
        // 也可以直接传进控件BP进InteractionButtonWidget，也可以直接根据蓝图路径代码创建Widget，再通过反射的方法FindByName，这样就不用传进类再创建了
        InteractionButtonWidget = CreateWidget<UUserWidget>(GetWorld(), InteractionButtonWidgetClass);

        if (InteractionButtonWidget)
        {
            // 将 Widget 添加到视口
            // InteractionButtonWidget->AddToViewport();

            // 初始隐藏按钮
            // InteractionButtonWidget->SetVisibility(ESlateVisibility::Hidden);

            // 获取按钮组件并绑定事件
            UButton* InteractionButton = Cast<UButton>(InteractionButtonWidget->GetWidgetFromName(TEXT("InteractionButton")));
            if (InteractionButton)
            {
                InteractionButton->OnClicked.AddDynamic(this, &AUIActor::OnInteractionButtonClicked);
            }

            
        }
    }

方法二
