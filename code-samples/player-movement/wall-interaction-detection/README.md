# Wall Interaction Detection

This excerpt comes from the player movement system. The player could interact with walls in several ways, including ledge grabs, wall climbs, wall scrapes, and wall runs. The goal of this logic was to identify which wall interaction was currently valid without requiring designer-placed volumes or special wall actors.

The system uses forward traces to detect walls and ledges, side traces to detect wall-run opportunities, and movement/input state checks to decide which action should occur. The result was a flexible movement system that let the player interact with ordinary level geometry. If I revisited this system today, I would likely split the detection, decision-making, and transition logic into separate helper functions or a dedicated wall interaction component.

This sample demonstrates:

- Trace-based ledge detection
- Wall climb and wall scrape validation
- Wall run detection on either side of the player
- Input direction checks using dot products
- Movement state gating
- Buffered jump support during wall interactions

```cpp
void APlayerChar::TryWallStuff()
{
    if (bHitstun || bActionLock) return;

    bValidWallClimb = false;
    bValidWallRun = false;

    // Two forward traces are used for ledge and wall detection.
    // The second trace is higher and shorter, allowing the player to distinguish
    // between a full wall and a reachable ledge.
    const bool bFirstTrace = LedgeFirstTrace(LedgeGrabRange);
    const bool bSecondTrace = LedgeSecondTrace(LedgeGrabRange);

    if (MovementState == EFrogMovementState::WALLCLIMBING)
    {
        if (!(bFirstTrace && bSecondTrace))
        {
            EndWallClimb(false);
            StartLedgeGrab();
            return;
        }
    }
    else if (MovementState == EFrogMovementState::WALLSCRAPING)
    {
        const bool bStillFacingWall = FVector::DotProduct(
            GetFakeInputVector(),
            LedgeGrabHitResult.Normal
        ) <= 0.f;

        if (!(bFirstTrace && bSecondTrace) || !bStillFacingWall)
        {
            SetMovementState(EFrogMovementState::STANDARD);
        }
    }

    if (bFirstTrace)
    {
        if (bSecondTrace)
        {
            const bool bInputTowardWall = FVector::DotProduct(
                GetFakeInputVector(),
                LedgeGrabHitResult.Normal
            ) < -0.6f;

            if (bInputTowardWall)
            {
                if (bCanWallClimb && MovementState != EFrogMovementState::DIVING)
                {
                    bValidWallClimb = true;

                    const bool bShouldStartScrape =
                        MovementState != EFrogMovementState::WALLCLIMBING &&
                        MovementState != EFrogMovementState::WALLSCRAPING &&
                        GetVelocity().Z < 0.f;

                    if (bShouldStartScrape)
                    {
                        StartWallScrape(false);
                    }

                    if (bJumpBufferActive)
                    {
                        Jump();
                    }

                    return;
                }
                else if (MovementState != EFrogMovementState::WALLCLIMBING &&
                         MovementState != EFrogMovementState::WALLSCRAPING &&
                         MovementState != EFrogMovementState::DIVING &&
                         GetVelocity().Z <= 0.f)
                {
                    StartWallScrape(false);
                }

                return;
            }
        }
        else
        {
            const bool bInputTowardLedge = FVector::DotProduct(
                GetLastMovementInputVector(),
                LedgeGrabHitResult.Normal
            ) < -0.3f;

            if (bInputTowardLedge)
            {
                StartLedgeGrab();
            }
        }
    }

    const FVector StartLoc = GetActorLocation();

    if (MovementState == EFrogMovementState::WALLRUNNING)
    {
        const FVector WallDirection = bWallOnRightSide
            ? GetActorRightVector()
            : -GetActorRightVector();

        if (!CapsuleTraceHitsStatic(StartLoc, StartLoc + WallDirection * WallRunRange, 25.f))
        {
            StopWallRun();
        }

        return;
    }

    if (MovementState == EFrogMovementState::HANGING || !bWallRunUnlocked)
    {
        return;
    }

    if (CapsuleTraceHitsStatic(StartLoc, StartLoc + GetActorRightVector() * WallRunRange, 25.f))
    {
        bValidWallRun = true;
        bWallOnRightSide = true;
    }
    else if (CapsuleTraceHitsStatic(StartLoc, StartLoc - GetActorRightVector() * WallRunRange, 25.f))
    {
        bValidWallRun = true;
        bWallOnRightSide = false;
    }
}
```

```cpp
bool APlayerChar::LedgeFirstTrace(float TraceLength)
{
    FVector StartLoc = GetActorLocation();
    StartLoc.Z += LedgeGrabVerticalAdjust;

    if (MovementState == EFrogMovementState::WALLCLIMBING)
    {
        StartLoc.Z += 100.f;
    }

    const FVector EndLoc = StartLoc + GetActorForwardVector() * TraceLength;

    FCollisionQueryParams CollisionParams;
    FHitResult NewHit;

    const bool bHit = GetWorld()->LineTraceSingleByObjectType(
        NewHit,
        StartLoc,
        EndLoc,
        ECC_WorldStatic,
        CollisionParams
    );

    if (bHit)
    {
        LedgeGrabHitResult = NewHit;
        LastValidLedgeGrabHitResult = NewHit;
        bHasLastValidLedgeGrabHit = true;
    }

    return bHit;
}

bool APlayerChar::LedgeSecondTrace(float TraceLength)
{
    FVector StartLoc = GetActorLocation();
    StartLoc.Z += LedgeGrabVerticalAdjust;
    StartLoc.Z += 50.f;

    const FVector EndLoc = StartLoc + GetActorForwardVector() * TraceLength;

    FCollisionQueryParams CollisionParams;
    FHitResult NewResult;

    return GetWorld()->LineTraceSingleByObjectType(
        NewResult,
        StartLoc,
        EndLoc,
        ECC_WorldStatic,
        CollisionParams
    );
}
```

## Notes

This sample is representative rather than a full movement component. The original player class also handled the animation, timers, transitions, and feedback associated with each wall action. For presentation, the excerpt focuses on detection and state selection.
