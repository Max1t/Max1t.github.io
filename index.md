## Helo

Portfolio 

### Code?

Code snippet test

<details>
<summary>Code snippet</summary>
{% highlight ruby %}
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
{% endhighlight %}
</details>
