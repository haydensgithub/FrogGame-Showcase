# Boss Melee Selection

This excerpt comes from the boss enemy controller. The goal of this logic was to choose a melee response based on the player's position relative to the boss.

The boss had several melee options, but the correct attack depended heavily on player placement. A player standing in front of the boss should be handled differently from a player jumping above it or slipping behind it. This made the boss feel more aware of the player and prevented the encounter from collapsing into a single repeated melee attack.

This sample demonstrates:

- Relative position checks
- Dot product use for detecting whether the player is behind the boss
- Verticality-aware attack selection
- Simple readable combat decision logic

```cpp
void AMommaRollController::MeleeAttack()
{
    const FVector ToPlayer = PlayerPointer->GetActorLocation() - MommaPointer->GetActorLocation();

    const bool bAbove = PlayerPointer->GetActorLocation().Z > MommaPointer->GetActorLocation().Z;
    const bool bDirectlyAbove = bAbove && FVector(ToPlayer.X, ToPlayer.Y, 0.f).SizeSquared() < 100.f;
    const bool bBehind = FVector::DotProduct(
        MommaPointer->GetActorForwardVector(),
        ToPlayer.GetSafeNormal()
    ) < -0.4f;

    if (bBehind || bDirectlyAbove)
    {
        MommaPointer->TailSwipe();
    }
    else if (bAbove)
    {
        MommaPointer->AirSwipe(false);
    }
    else
    {
        MommaPointer->Uppercut();
    }
}
```

## Notes

This is a small excerpt, but it shows the kind of spatial reasoning used throughout the enemy AI. Rather than choosing melee attacks randomly, the controller uses the player's current position to select the attack that best matches the situation.
