# Minion Spawn Selection

This excerpt comes from the boss enemy actor. The boss could summon smaller enemies during combat, but those enemies needed to appear on valid ground rather than spawning inside walls, in midair, or on steep surfaces.

The function makes several randomized spawn attempts around the boss. For each attempt, it traces downward from above the candidate location, checks whether the trace hit valid ground, verifies that the surface is reasonably flat, and then spawns the minion slightly above the impact point.

This sample demonstrates:

- Randomized spawn selection
- Line traces for ground detection
- Surface normal validation
- Safe early returns
- Controlled actor spawning in Unreal Engine

```cpp
void AMommaRoll::SpawnBabyRoll()
{
    if (!GetWorld()) return;

    const int32 MaxAttempts = 12;
    const float TraceHeight = 500.f;
    const float MinUpDot = 0.7f;

    for (int32 i = 0; i < MaxAttempts; i++)
    {
        const float Angle = FMath::RandRange(0.f, 2 * PI);
        const float Radius = FMath::RandRange(200.f, SpawnRadius);

        const FVector Offset = FVector(FMath::Cos(Angle), FMath::Sin(Angle), 0.f) * Radius;
        const FVector Start = GetActorLocation() + Offset + FVector(0.f, 0.f, TraceHeight);
        const FVector End = Start - FVector(0.f, 0.f, TraceHeight * 2.f);

        FHitResult Hit;
        FCollisionQueryParams Params;
        Params.AddIgnoredActor(this);

        const bool bHit = GetWorld()->LineTraceSingleByChannel(
            Hit,
            Start,
            End,
            ECC_Visibility,
            Params
        );

        const bool bValidSurface = bHit &&
            FVector::DotProduct(Hit.ImpactNormal, FVector::UpVector) >= MinUpDot;

        if (bValidSurface)
        {
            const FVector SpawnLocation = Hit.ImpactPoint + FVector(0.f, 0.f, 10.f);
            const FRotator SpawnRotation = FRotator::ZeroRotator;

            GetWorld()->SpawnActor<ABabyRoll>(BabyRollBP, SpawnLocation, SpawnRotation);
            return;
        }
    }
}
```

## Notes

This logic was written to make enemy summoning feel reliable without requiring hand-placed spawn points. If no valid location is found after the allowed attempts, the function simply fails without spawning anything, avoiding bad spawns during combat.
