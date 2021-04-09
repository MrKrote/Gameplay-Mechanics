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
**SpawnOurPawn_Implementation(UClass* ToSpawn, const FVector& Location)**
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






