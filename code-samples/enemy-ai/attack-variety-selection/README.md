# Attack Variety Selection

This excerpt comes from an enemy controller responsible for choosing between two attack plans. The enemy could either perform a jump attack or a double slash. Because purely random selection can create repetitive behavior, the controller tracks the previously selected attack and prevents the same move from being chosen too many times in a row.

The controller also supports delayed attacks after pathing into range. This helped the enemy avoid instantly attacking the player the moment it reached the target, which made the behavior feel more natural and readable.

This sample demonstrates:

- Enemy attack planning
- Avoiding repeated move selection
- Simple state-driven AI behavior
- Delayed attack timing through Unreal timers
- Movement-to-attack transitions

```cpp
void AIsopodNinjaController::ChooseDesiredAttack()
{
    // Do not change the planned attack while an attack delay is already active.
    if (bAttackDelayActive)
    {
        return;
    }

    if (NinjaPointer->AnimState != EIsopodNinjaAnimState::STANDARD)
    {
        return;
    }

    const int32 Rand = FMath::RandRange(0, 1);
    DesiredAttack = (Rand == 0)
        ? EIsopodNinjaDesiredAttack::JUMPATTACK
        : EIsopodNinjaDesiredAttack::DOUBLESLASH;

    if (DesiredAttack == PreviousAttack)
    {
        RepeatedMoveCount++;
    }
    else
    {
        RepeatedMoveCount = 0;
    }

    // If this would be the third use in a row, force the other attack instead.
    if (RepeatedMoveCount > 1)
    {
        DesiredAttack = (DesiredAttack == EIsopodNinjaDesiredAttack::JUMPATTACK)
            ? EIsopodNinjaDesiredAttack::DOUBLESLASH
            : EIsopodNinjaDesiredAttack::JUMPATTACK;

        RepeatedMoveCount = 0;
    }
}
```

```cpp
void AIsopodNinjaController::JumpAttack()
{
    const bool bJumpLeft = FMath::RandBool();
    const float AngleDeg = bJumpLeft ? 90.f : -90.f;

    NinjaPointer->AnimState = EIsopodNinjaAnimState::FLIPPING;
    PreviousAttack = EIsopodNinjaDesiredAttack::JUMPATTACK;
    DesiredAttack = EIsopodNinjaDesiredAttack::NONE;
    bAttackDelayActive = false;
    bRanToTarget = false;
    NinjaPointer->bFreezeController = true;

    const FVector LandingPoint = GetCircleJumpPoint(
        PlayerPointer->GetActorLocation(),
        JumpAttackRange * 1.4f,
        AngleDeg
    );

    FVector JumpDir = LandingPoint - NinjaPointer->GetActorLocation();
    JumpDir.Z = 0.f;
    JumpDir.Normalize();

    const FVector Launch = JumpDir * 1400.f + FVector(0.f, 0.f, 1000.f);
    EnemyPointer->LaunchCharacter(Launch, true, true);
    EnemyPointer->RotateToLocation(LandingPoint, 0.1f);
}
```

## Notes

This enemy was intended to be aggressive without feeling completely random. The attack selection is still simple, but the anti-repetition rule gives the enemy more deliberate pacing and prevents the fight from feeling stuck on one behavior.
