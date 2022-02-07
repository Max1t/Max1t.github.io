# Airships

Another Student project. A demo with programmer graphics but functional gameplay.

![Image](/assets/airship.png)

## Features

- Traversable map inspired by games like Faster Than Light
- Real time combat against an AI
- Battles utilize an match 3 board to gather resources in battle. AI looks for matches simultaneously
- Match bombs on the board to dmg the enemy and use gathered resources to use items to effect the board or cause other effects
- Control crew members to move between different quarters to boost a variety of ship systems

## Some Code snippets

Full source is in a private repository.
Here are some code snippets from various things i worked on.

<details>
<summary>A weapon using a baseclass that affects the match3 board, destroying a row of blocks</summary>


{% highlight csharp %}

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

{% endhighlight %}
</details>

<details>
<summary>Here is finding if there is a legal move left on the board.</summary>


{% highlight csharp %}

    public GameObject[,] FindAllInOrder(GameObject startingBlock)
    {
        GameObject[,] tempArray = new GameObject[8, 8];
        tempArray[0, 0] = startingBlock;
        for (int x = 0; x < 8; x++)
        {
            for (int y = 0; y < 8; y++)
            {
                if (x == 7 && y == 7)
                {
                    continue;
                }
                if (y == 7)
                {
                    if (matches.GetBlockRight(tempArray[x, 0]) == null)
                    {
                        nullCheck = true;
                        return tempArray;
                    }
                    tempArray[x + 1, 0] = matches.GetBlockRight(tempArray[x, 0]);
                }
                else
                {
                    if (matches.GetBlockUp(tempArray[x, y]) == null)
                    {
                        nullCheck = true;
                        return tempArray;
                    }
                    tempArray[x, y + 1] = matches.GetBlockUp(tempArray[x, y]);
                }
            }
        }
        return tempArray;
    }

    // Tests all possible moves for a block for a match. If a match would occur it returns true
    public bool TestBlockMatches(GameObject[,] GoArray, int index_x, int index_y)
    {
        string targetTag = GoArray[index_x, index_y].tag;
        // Method does not move any blocks in the game nor in the array just looks at the blocks  
        // at the correct indexes for array given as parameter
        // Test UP
        if (index_y + 1 < 8)
        {
            int howManyUpDown = 1;
            int howManyLeftRight = 1;
            int simulatedPositionIndex_y = index_y + 1;
            // Up
            for (int i = 1; i < 8 - simulatedPositionIndex_y; i++)
            {
                if (GoArray[index_x, simulatedPositionIndex_y + i].tag == targetTag)
                {
                    howManyUpDown++;
                }
                else break;
            }
            // Left
            for (int i = 1; i < index_x + 1; i++)
            {
                if (GoArray[index_x - i, simulatedPositionIndex_y].tag == targetTag)
                {
                    howManyLeftRight++;
                }
                else break;
            }
            //Right
            for (int i = 1; i < 8 - index_x; i++)
            {
                if (GoArray[index_x + i, simulatedPositionIndex_y].tag == targetTag)
                {
                    howManyLeftRight++;
                }
                else break;
            }
            if (howManyLeftRight >= 3 || howManyUpDown >= 3) return true;
        }


        // Test DOWN
        if (index_y - 1 > -1)
        {
            int howManyUpDown = 1;
            int howManyLeftRight = 1;
            int simulatedPositionIndex_y = index_y - 1;
            // Down
            for (int i = 1; i < simulatedPositionIndex_y; i++)
            {
                if (GoArray[index_x, simulatedPositionIndex_y - i].tag == targetTag)
                {
                    howManyUpDown++;
                }
                else break;
            }
            // Left
            for (int i = 1; i < index_x + 1; i++)
            {
                if (GoArray[index_x - i, simulatedPositionIndex_y].tag == targetTag)
                {
                    howManyLeftRight++;
                }
                else break;
            }
            for (int i = 1; i < 8 - index_x; i++)
            {
                if (GoArray[index_x + i, simulatedPositionIndex_y].tag == targetTag)
                {
                    howManyLeftRight++;
                }
                else break;
            }
            if (howManyLeftRight >= 3 || howManyUpDown >= 3) return true;
        }


        // Test LEFT
        if (index_x - 1 > -1)
        {
            int howManyUpDown = 1;
            int howManyLeftRight = 1;
            int simulatedPositionIndex_x = index_x - 1;
            //Up
            for (int i = 1; i < 8 - index_y; i++)
            {
                if (GoArray[simulatedPositionIndex_x, index_y + i].tag == targetTag)
                {
                    howManyUpDown++;
                }
                else break;
            }
            //Down
            for (int i = 1; i < index_y + 1; i++)
            {
                if (GoArray[simulatedPositionIndex_x, index_y - i].tag == targetTag)
                {
                    howManyUpDown++;
                }
                else break;
            }
            // Left
            for (int i = 1; i < simulatedPositionIndex_x + 1; i++)
            {
                if (GoArray[simulatedPositionIndex_x - i, index_y].tag == targetTag)
                {
                    howManyLeftRight++;
                }
                else break;
            }
            if (howManyLeftRight >= 3 || howManyUpDown >= 3) return true;
        }


        // Test RIGHT
        if (index_x + 1 < 8)
        {
            int howManyUpDown = 1;
            int howManyLeftRight = 1;
            int simulatedPositionIndex_x = index_x + 1;
            // Up
            for (int i = 1; i < 8 - index_y; i++)
            {
                if (GoArray[simulatedPositionIndex_x, index_y + i].tag == targetTag)
                {
                    howManyUpDown++;
                }
                else break;
            }
            // Down
            for (int i = 1; i < index_y + 1; i++)
            {
                if (GoArray[simulatedPositionIndex_x, index_y - i].tag == targetTag)
                {
                    howManyUpDown++;
                }
                else break;
            }
            // Right
            for (int i = 1; i < 8 - simulatedPositionIndex_x; i++)
            {
                if (GoArray[simulatedPositionIndex_x + i, index_y].tag == targetTag)
                {
                    howManyLeftRight++;
                }
                else break;
            }
            if (howManyLeftRight >= 3 || howManyUpDown >= 3) return true;
        }

        return false;
    }
}

{% endhighlight %}
</details>

<details>
<summary>Health and steam tracking and UI animation</summary>


{% highlight csharp %}


    void Start()
    {
        //AirshipStats.airshipCurrentHealth = AirshipStats.airshipMaxHealth;
        AirshipStats.currentSteam = 0;
        UpdateUI();
    }

    void Update()
    {
        LerpHealth();
        LerpSteam();
    }

    // Healthin muuttaminen
    public void TakeDamage(float damageTaken)
    {
        AirshipStats.airshipCurrentHealth -= damageTaken;
        previousHealth = healthFill.fillAmount * AirshipStats.airshipMaxHealth;
        currentHealthLerpTime = 0;
        if (AirshipStats.airshipCurrentHealth <= 0)
        {
            // ded
            AirshipStats.airshipCurrentHealth = 0;
            AirshipStats.battlePause = true; //ei toimi?
        }
        UpdateUI();
    }

    // Steamin käyttö VIP
    public void UseSteam(float steamUsed)
    {
        AirshipStats.currentSteam -= steamUsed;
        previousSteam = steamFill.fillAmount * AirshipStats.maxSteam;
        currentSteamLerpTime = 0;
        if (AirshipStats.currentSteam < 0)
        {
            //sumffing
            AirshipStats.currentSteam = 0;
        }
        UpdateUI();
    }

    private void LerpHealth()                           // Health Bar animaatio
    {
        if (currentHealthLerpTime > healthLerpTime)
        {
            previousHealth = AirshipStats.airshipCurrentHealth;

            currentHealthLerpTime = 0;
        }
        else
        {
            currentHealthLerpTime += Time.deltaTime;
        }

        float t = currentHealthLerpTime / healthLerpTime;
        t = Mathf.Sin(t * Mathf.PI * 0.5f);
        healthFill.fillAmount = Mathf.Lerp(previousHealth / AirshipStats.airshipMaxHealth, AirshipStats.airshipCurrentHealth 
                                                            / AirshipStats.airshipMaxHealth, t);
    }

    private void LerpSteam()                            // Steam Bar animaatio
    {
        if (currentSteamLerpTime > steamLerpTime)
        {
            previousSteam = AirshipStats.currentSteam;

            currentSteamLerpTime = 0;
        }
        else
        {
            currentSteamLerpTime += Time.deltaTime;
        }

        float t = currentSteamLerpTime / steamLerpTime;
        t = Mathf.Sin(t * Mathf.PI * 0.5f);
        steamFill.fillAmount = Mathf.Lerp(previousSteam / AirshipStats.maxSteam, AirshipStats.currentSteam 
                                                                                / AirshipStats.maxSteam, t);
    }

   public void UpdateUI()                                     // Päivittää UI tekstit
    {
        healthText.text = AirshipStats.airshipCurrentHealth.ToString();
        steamText.text = AirshipStats.currentSteam.ToString();
    }


{% endhighlight %}
</details>

<details>
<summary>The full inventory system for buying and equipping items and guns</summary>

{% highlight csharp %}

public class Inventory : MonoBehaviour
{
    public static Inventory instance;


    public GunBase defaultGun1;
    public GunBase defaultGun2;

    public delegate void OnInventoryChanged();
    public OnInventoryChanged OnInventoryChangedCallback;

    public List<GunBase> equippedItems = new List<GunBase>();
    public List<GunBase> cargoItems = new List<GunBase>();
    public int cargoSpace = 12;
    public int equipmentSpace = 4;

    public InstanceShipScene crew;


    public List<GunBase> allItems = new List<GunBase>();
    public List<GunBase> shopItems = new List<GunBase>();

    void Awake()
    {
        crew = FindObjectOfType<InstanceShipScene>();
        if (instance != null) return;
        instance = this;

        int[] testArray = new int[allItems.Count];
        for (int i = 0; i < allItems.Count; i++)
        {
            testArray[i] = allItems[i].itemCode;
        }
        if (testArray.GroupBy(x => x).Any(g => g.Count() > 1)) // Test for duplicate item codes
            throw new Exception("Duplicate item codes");       // Saving/loading depends on no duplicates

        testArray = new int[0];

        EquipItem(defaultGun1);
        AddItemToCargo(defaultGun2);



    }

    public void addCrew()
    {
        if (AirshipStats.credits >= 500f)
        {
            if (AirshipStats.howManyNewCrew < 4)
            {
                AudioManager.instance.Play("Osto");
                AirshipStats.howManyNewCrew++;
                crew.InstanceTheCrewFirst();
            }
            if (OnInventoryChangedCallback != null) OnInventoryChangedCallback.Invoke();
        }
    }


    public bool CargoToEquip(GunBase item)
    {
        if (EquipItem(item))
        {
            if (OnInventoryChangedCallback != null) OnInventoryChangedCallback.Invoke();
            RemoveItemFromCargo(item);
            return true;
        }
        else return false;
    }

    public bool EquipToCargo(GunBase item)
    {
        if (AddItemToCargo(item))
        {
            if (OnInventoryChangedCallback != null) OnInventoryChangedCallback.Invoke();
            UnequipItem(item);
            return true;
        }
        else return false;

    }

    public bool AddItemToCargo(GunBase item)
    {
        if (cargoItems.Count >= cargoSpace) return false;
        cargoItems.Add(item);
        if (OnInventoryChangedCallback != null) OnInventoryChangedCallback.Invoke();
        return true;
    }

    public void RemoveItemFromCargo(GunBase item)
    {
        cargoItems.Remove(item);
        if (OnInventoryChangedCallback != null) OnInventoryChangedCallback.Invoke();
    }

    public bool EquipItem(GunBase item)
    {
        if (equippedItems.Count >= equipmentSpace) return false;
        equippedItems.Add(item);
        if (OnInventoryChangedCallback != null) OnInventoryChangedCallback.Invoke();
        return true;
    }

    public void UnequipItem(GunBase item)
    {
        equippedItems.Remove(item);
        if (OnInventoryChangedCallback != null) OnInventoryChangedCallback.Invoke();
    }


    public void PopulateShop()
    {
        foreach (var item in allItems)
        {
            if (UnityEngine.Random.Range(0, 2) == 1)
                shopItems.Add(item);
        }
    }

    public void AddToShop(GunBase item)
    {
        shopItems.Add(item);
        if (OnInventoryChangedCallback != null) OnInventoryChangedCallback.Invoke();
    }

    public void RemoveFromShop(GunBase item)
    {
        shopItems.Remove(item);
        if (OnInventoryChangedCallback != null) OnInventoryChangedCallback.Invoke();
    }

    public void SellItem(GunBase item)
    {

        RemoveItemFromCargo(item);
        AddToShop(item);
        AirshipStats.credits += item.sellValue;
        Debug.Log(AirshipStats.credits);
    }

    public bool BuyItem(GunBase item)
    {
        if (AirshipStats.credits > item.buyValue)
        {
            AddItemToCargo(item);
            RemoveFromShop(item);
            AudioManager.instance.Play("Osto");
            AirshipStats.credits -= item.buyValue;
            Debug.Log(AirshipStats.credits);
            return true;
        }
        return false;
    }


    public int[] SaveCargoData()
    {
        int[] temp = new int[cargoItems.Count];
        for (int i = 0; i < cargoItems.Count; i++)
        {
            temp[i] = cargoItems[i].itemCode;
        }
        return temp;
    }

    public void LoadCargoData(int[] cargoData)
    {
        cargoItems.Remove(defaultGun2);
        foreach (int itemCode in cargoData)
        {
            GunBase tempItem = null;
            while (tempItem == null)
            {
                foreach (GunBase item in allItems)
                {
                    if (item.itemCode == itemCode)
                    {
                        tempItem = item;
                    }
                }
            }
            cargoItems.Add(tempItem);
        }
    }

    public int[] SaveEquippedData()
    {
        int[] temp = new int[equippedItems.Count];
        for (int i = 0; i < equippedItems.Count; i++)
        {
            temp[i] = equippedItems[i].itemCode;
        }
        return temp;
    }

    public void LoadEquippedData(int[] equippedData)
    {
        equippedItems.Remove(defaultGun1);
        foreach (int itemCode in equippedData)
        {
            GunBase tempItem = null;
            while (tempItem == null)
            {
                foreach (GunBase item in allItems)
                {
                    if (item.itemCode == itemCode)
                    {
                        tempItem = item;
                    }
                }
            }
            equippedItems.Add(tempItem);
        }
    }

}

{% endhighlight %}
</details>