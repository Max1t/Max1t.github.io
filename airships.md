# Airships

![Image](/assets/airship.png)

## Features

- Traversable map inspired by games like Faster Than Light
- Real time combat against an AI
- Battles utilize an match 3 board to gather resources in battle. AI looks for matches simultaneously
- Match bombs on the board to dmg the enemy and use gathered resources to use items to effect the board or cause other effects
- Control crew members to move between different quarters to boost a variety of ship systems

## Some Code snippets

<details>
<summary>Code snippet</summary>

```csharp
public class GunDestroyRow : GunBase
{
    public override void OnButtonDown()
    {
        matches.playersTurn = true;
        if (Coroutine.choosingBlock == true)
        {
            Coroutine.choosingBlock = false;
            return;
        }
        swap.ResetClicks();
        if (blockStorage.checkBlocksFromStorageMultipleColor(colorsToUse, amountOfColorsToUse))
        {
            Coroutine.choosingBlock = true;
            Coroutine.StartChoosing(this, ChooseBlockCoroutine.Ability.ROW, soundClip);
        }
    }
}
```

</details>
