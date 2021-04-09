# Gameplay-Mechanics

> Floor switch
> 

**.H**
```
/** Overlap volume for functionality to be triggered */
UPROPERTY(VisibleAnywhere,BlueprintReadWrite,Category = "FloorSwitch")
class UBoxComponent* TriggerBox;
 
/** Switch for the character to step on */
UPROPERTY(VisibleAnywhere,BlueprintReadWrite,Category = "FloorSwitch")
class UStaticMeshComponent* FloorSwitch;
 
/** Door to move when the floor switch is stepped on */
UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "FloorSwitch")
UStaticMeshComponent* Door;
UFUNCTION()
void OnOverlapBegin(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex, bool bFromSweep, const FHitResult& SweepResult);
UFUNCTION()
void OnOverlapEnd(UPrimitiveComponent* OverlappedComponent, AActor* OtherActor, UPrimitiveComponent* OtherComp, int32 OtherBodyIndex);

UFUNCTION(BlueprintImplementableEvent, Category = "FloorSwitch")
void RaiseDoor();
UFUNCTION(BlueprintImplementableEvent, Category = "FloorSwitch")
void LowerDoor();
UFUNCTION(BlueprintImplementableEvent, Category = "FloorSwitch")
void RaiseFloorSwitch();
UFUNCTION(BlueprintImplementableEvent, Category = "FloorSwitch")
void LowerFloorSwitch();
UFUNCTION(BlueprintCallable, Category = "FloorSwitch")
void UpdateDoorLocation(float Z);
UFUNCTION(BlueprintCallable, Category = "FloorSwitch")
void UpdateFloorSwitchLocation(float Z);

FTimerHandle SwitchHandle;
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "FloorSwitch")
float SwitchTime;
bool bCharacterOnSwitch;
void CloseDoor();
```
**.CPP**
**Constructor**
```
RootComponent = TriggerBox;
TriggerBox->SetCollisionEnabled(ECollisionEnabled::QueryOnly); 
 
TriggerBox->SetCollisionObjectType(ECollisionChannel::ECC_WorldStatic);
 
TriggerBox->SetCollisionResponseToAllChannels(ECollisionResponse::ECR_Ignore);
 
TriggerBox->SetCollisionResponseToChannel(ECollisionChannel::ECC_Pawn, ECollisionResponse::ECR_Overlap);
 
TriggerBox->SetBoxExtent(FVector(62.0f, 62.0f, 32.f));
 
FloorSwitch = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("FloorSwitch"));
FloorSwitch->SetupAttachment(GetRootComponent());
 
Door = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Door"));
Door->SetupAttachment(GetRootComponent());

SwitchTime = 2.0f;
bCharacterOnSwitch = false;
```

**BeginPlay ()**
```
TriggerBox->OnComponentBeginOverlap.AddDynamic(this, &AFloorSwitch::OnOverlapBegin);
TriggerBox->OnComponentEndOverlap.AddDynamic(this, &AFloorSwitch::OnOverlapEnd);


InitialDoorLocation = Door->GetComponentLocation();
InitialSwitchLocation = FloorSwitch->GetComponentLocation();
```


**OnOverlapBegin()**
```
UE_LOG(LogTemp, Warning, TEXT("Overlap Begin."));
if (!bCharacterOnSwitch) 
{
bCharacterOnSwitch = true;
}
RaiseDoor();
LowerFloorSwitch();
```

**OnOverlapEnd()**
```
UE_LOG(LogTemp, Warning, TEXT("Overlap End."));
if (bCharacterOnSwitch)
{
bCharacterOnSwitch = false;
}
GetWorldTimerManager().SetTimer(SwitchHandle, this, &AFloorSwitch::CloseDoor, SwitchTime);
```

**UpdateDoorLocation(float Z)**
```
FVector NewLocation = InitialDoorLocation;
NewLocation.Z += Z;
Door->SetWorldLocation(NewLocation);
```

**UpdateFloorSwitchLocation(float Z)**
```
FVector NewLocation = InitialSwitchLocation;
NewLocation.Z += Z;
FloorSwitch->SetWorldLocation(NewLocation);
```

**CloseDoor()**
```
if (!bCharacterOnSwitch)
{
LowerDoor();
RaiseFloorSwitch();
}
```

**Blueprint SIDE**

Event Raise Door(Play) + Event Lower Door (Reverse) + Get World Delta Seconds (New Time) -  add Timeline (Update + Door Elevation (Float Track) ) - Update Door Location

Event Lower Floor Switch (Play) + Event Raise Floor Switch (Reverse) + Get World Delta Seconds (New Time) -  add Timeline (Update + Floor Switch Elevation (Float Track) ) - Update Floor Switch Location

&nbsp;

> #1 Spawn Critter on random position with effect - SpawnVolume C++ (Actor) Class
> 

**.H**
```
UPROPERTY(VisibleAnywhere,BlueprintReadOnly,Category = "Spawning")
class UBoxComponent* SpawningBox;
 
UPROPERTY(EditAnywhere,BlueprintReadOnly, Category = "Spawning")
TSubclassOf<class ACritter> PawnToSpawn;
 
UFUNCTION(BlueprintPure, Category = "Spawning")
FVector GetSpawnPoint();
 
UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = "Spawning")
void SpawnOurPawn(UClass* ToSpawn, const FVector& Location);
```
**.CPP**
**Constructor**
```
SpawningBox = CreateDefaultSubobject<UBoxComponent>(TEXT("SpanningBox"));
```
**GetSpawnPoint()**
```
FVector Extent = SpawningBox->GetScaledBoxExtent();
FVector Origin = SpawningBox->GetComponentLocation();
FVector Point = UKismetMathLibrary::RandomPointInBoundingBox(Origin, Extent);
return Point;
```
**SpawnOurPawn_Implementation(UClass* ToSpawn,const FVector& Location)**
```
if (ToSpawn)
{
UWorld* World = GetWorld();
FActorSpawnParameters SpawnParams;
if (World)
{
ACritter* CritterSpawn = World->SpawnActor<ACritter>(ToSpawn,Location,FRotator(0.f), SpawnParams);
}}
```

SpawnVolume_BP- SpawnVolume_BP(self) - Pawn to Spawn set to Critter_BP

BP SIDE 

Event BeginPlay + Pawn to Spawn ( Getter ) + Get Spawn Point ( Getter ) - Spawn Our Pawn ( BlueprintCallable BLUE )

Event Spawn Our Pawn ( To Spawn + Location) - Parent : Spawn Our Pawn ( To Spawn + Location)   ( to create it:  right click -> add parent )

Event Spawn Our Pawn (Execution) - Spawn Ermitter at Location - Parent: Spawn Our Pawn (Execution )

&nbsp;

> Moving Platform
> 

**.H**
```
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Platform")
class UStaticMeshComponent* Mesh;
UPROPERTY(EditAnywhere)
FVector StartPoint;
UPROPERTY(EditAnywhere, meta = (MakeEditWidget = "true"))
FVector EndPoint;
UPROPERTY(VisibleAnywhere,BlueprintReadOnly,Category = "Platform")
float InterpSpeed;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Platform")
FTimerHandle InterpTimer;
UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Platform")
bool bInterping;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Platform")
float InterpTime;
UPROPERTY(EditAnywhere, BlueprintReadOnly, Category = "Platform")
float Distance;
 
void ToggleInterping();
void SwapVectors(FVector& VecOne, FVector& VecTwo);
```
**.CPP**
**Constructor**
```
Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh"));
RootComponent = Mesh;
 
StartPoint = FVector(0.f);
EndPoint = FVector(0.f);
InterpSpeed = 4.0f;
InterpTime = 2.0f;
bInterping = false;
```
**BeginPlay()**
```
StartPoint = GetActorLocation();
EndPoint += StartPoint;
GetWorldTimerManager().SetTimer(InterpTimer, this, &AFloatingPlatform::ToggleInterping, InterpTime);
Distance = (EndPoint - StartPoint).Size();
```
**Tick ()**
```
if (bInterping)
{
FVector CurrentLocation = GetActorLocation();
FVector Interp = FMath::VInterpTo(CurrentLocation, EndPoint, DeltaTime, InterpSpeed);
SetActorLocation(Interp);
float DistanceTraveled = (GetActorLocation() - StartPoint).Size();
if (Distance - DistanceTraveled <= 1.0f)
{
ToggleInterping();
GetWorldTimerManager().SetTimer(InterpTimer, this, &AFloatingPlatform::ToggleInterping, InterpTime);
SwapVectors(StartPoint, EndPoint);
}}
```
**ToggleInterping()**
```
{
bInterping = !bInterping;
}
```
**SwapVectors(FVector& VecOne, FVector& VecTwo)**
```
{
FVector Temp = VecOne;
VecOne = VecTwo;
VecTwo = Temp;
}
```

&nbsp;

> Item + PickUP + Explosive
> 
**Item.h**
```
UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "Item | Collision")
class USphereComponent* CollisionVolume;
 
UPROPERTY(VisibleAnywhere, BlueprintReadWrite, Category = "Item | Mesh")
class UStaticMeshComponent* Mesh;
 
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item | Particles")
class UParticleSystemComponent* IdleParticleComponent;
 
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item | Particles")
class UParticleSystem* OverlapParticles;
 
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item | Sounds")
class USoundCue* OverlapSound;
 
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item | ItemProperties")
bool bRotate;
 
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Item | ItemProperties")
float RotationRate;
 
UFUNCTION()
virtual void OnOverlapBegin(PARAMETERS);
UFUNCTION()
virtual void OnOverlapEnd(PARAMETERS);
```
**Item.cpp**
**Constructor**
```
CollisionVolume = CreateDefaultSubobject<USphereComponent>(TEXT("CollisionVolume"));
RootComponent = CollisionVolume;
 
Mesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("Mesh"));
Mesh->SetupAttachment(GetRootComponent());
 
IdleParticleComponent = CreateDefaultSubobject<UParticleSystemComponent>(TEXT("IdleParticleComponent"));
IdleParticleComponent->SetupAttachment(GetRootComponent());
 
bRotate = false;
RotationRate = 45.0f;
```
**BeginPlay()**
```
CollisionVolume->OnComponentBeginOverlap.AddDynamic(this, &AItem::OnOverlapBegin);
CollisionVolume->OnComponentEndOverlap.AddDynamic(this, &AItem::OnOverlapEnd);
```
**Tick()**
```
if (bRotate)
{
FRotator Rotation = GetActorRotation();
Rotation.Yaw += DeltaTime * RotationRate;
SetActorRotation(Rotation);
}
```
**OnOverlapBegin ( PARAMETERS )**
```
UE_LOG(LogTemp, Warning, TEXT("Super::OnOverlapBegin"));
 
if (OverlapParticles)
{
UGameplayStatics::SpawnEmitterAtLocation(GetWorld(), OverlapParticles, GetActorLocation(), FRotator(0.f), true);
}
if (OverlapSound)
{
UGameplayStatics::PlaySound2D(this, OverlapSound);
}
Destroy();
```
**OnOverlapEnd ( PARAMETERS )**
```
UE_LOG(LogTemp, Warning, TEXT("Super::OnOverlapEnd"));
```

In Editor -> Create C++ Class derived from Item ( INHERIT )

1. Explosive

2. Pickup

**Explosive.h**
```
AExplosive();
virtual void OnOverlapBegin(PARAMETERS) override;
virtual void OnOverlapEnd(PARAMETERS) override;
```
**Explosive.cpp**
**OnOverlapBegin(PARAMETERS)**
```
Super::OnOverlapBegin(PARAMETERS);
UE_LOG;
```
**OnOverlapEnd(PARAMETERS)**
```
Super::OnOverlapBegin(PARAMETERS);
UE_LOG;
```
1.Static Mesh

2.IdleParticleComponent - Particles ( Template ) 

3.IdleParticleComponent - Sound

4._BP (Self) - Overlap Particles

&nbsp;

> HUD
> 

FirstProject.Build.cs -> " UMG "  + -> "Slate", "SlateCore"

1.Create a C++ PlayerController -> Create HUDOverlayAsset + HUDOverlay

2.Make a BP from PlayerController -> Set HUDOverlay Asset to HUDOverlay

3.GameMode - > Set Player Controller Class to MainPlayerController_BP

4.Create WidgetBlueprint (HUDOverlay , HealthBar , StaminaBar (Duplicate Healthbar)

5.HUDOverlay Widget Blueprint - > Set it ( Add HealthBar + StaminaBar to it)

6.HealthBar -> Set it -> Add a ProgressBar to it and Create Binding and give logic.
Get Percent + (Get)Ref To Main ( Make a variable in BP ) -> IS Valid? -> Return Node
Ref To Main -> Get Health + Get Max Health -> float / float -> Return Node
Get Player Pawn -> Cast To Main_BP -> (Set)Ref To Main -> Return Node

7.StaminaBar -> the same

