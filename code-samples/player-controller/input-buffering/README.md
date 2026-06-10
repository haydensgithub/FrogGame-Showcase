# Input Buffering

This excerpt comes from the player controller/character logic. The goal of the input buffer was to make movement and combat feel more responsive by remembering certain player inputs while the character was temporarily locked out of acting.

For example, if the player pressed jump or attack slightly before an action lock ended, the game could execute that input as soon as the lock cleared instead of dropping it. This made the controls feel more forgiving during fast combat and movement.

This sample demonstrates:

- Buffered input represented with an enum
- Separate handling for action locks and attack locks
- Responsive control feel during short lockout windows
- State validation before replaying stored input

```cpp
UENUM(BlueprintType)
enum class EBufferedInput : uint8
{
    NONE,
    JUMP,
    DASH,
    CROUCH,
    LIGHTATTACK,
};
```

```cpp
void APlayerChar::ReleaseActionLock()
{
    bActionLock = false;

    if (bRightClickHeld)
    {
        const bool bCanBlock =
            !GetCharacterMovement()->IsFalling() &&
            !bHitstun &&
            !bActionLock;

        if (bCanBlock)
        {
            InputBuffer = EBufferedInput::NONE;
            StartBlocking();
            return;
        }
    }

    if (InputBuffer == EBufferedInput::NONE)
    {
        return;
    }

    switch (InputBuffer)
    {
    case EBufferedInput::JUMP:
        Jump();
        InputBuffer = EBufferedInput::NONE;
        break;

    case EBufferedInput::CROUCH:
        if (bDownCommandHeld)
        {
            BeginDownCommand();
        }
        InputBuffer = EBufferedInput::NONE;
        break;

    case EBufferedInput::DASH:
        Dash();
        InputBuffer = EBufferedInput::NONE;
        break;

    default:
        break;
    }
}
```

```cpp
void APlayerChar::ReleaseAttackLock()
{
    bAttackLock = false;

    if (InputBuffer == EBufferedInput::NONE)
    {
        return;
    }

    switch (InputBuffer)
    {
    case EBufferedInput::LIGHTATTACK:
        if (HitstunState != EFrogHitstunState::NONE)
        {
            return;
        }

        PrimaryAttackPressed();
        InputBuffer = EBufferedInput::NONE;
        break;

    default:
        break;
    }
}
```

## Notes

This is a small system, but it is important for game feel. In a fast action game, controls can feel unresponsive if input is ignored during brief animation or movement locks. The buffer allowed selected actions to survive those short lockout windows while still respecting combat state checks.
