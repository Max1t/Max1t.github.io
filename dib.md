# Dibs

A project to create a feature for the game Dibs by Playstack.

![Image](/assets/dibs.png)

## Features

- A new way to interact with parcels on the game map. Items that are needed to reach some which you can aqquire from the ingame shop.
- New multiplayer focused "Parcel Giant" where people would gather parcels together for a larger prize.
- New Minigames to acquire parcels. Minigames are played in AR using the phones camera.
- A inventory system where you can buy different boosters and equipment for these new minigames.

## Some Code snippets

Full source is in a private repository.
Here are some code snippets from various things I worked on.


<details>
<summary> Class for handling a players inventory connected to an online database </summary>

{% highlight csharp %}
 
 public class PlayerInventory : MonoBehaviour
{

    private float _currency;
    public float Currency { get => _currency; set => _currency = value; }

    private PlayerGadgetInventory _gadgetInventory;
    public PlayerGadgetInventory GadgetInventory { get => _gadgetInventory; set => _gadgetInventory = value; }


    [SerializeField]
    private List<Parcel> _parcelInventory = new List<Parcel>();
    public List<Parcel> ParcelInventory { get => _parcelInventory; }

    private bool _snatchatron;
    public bool Snatchatron { get => _snatchatron; set => _snatchatron = value; }

    private int _parcelInventorySize = 10;

    [SerializeField]
    private PlayerInventoryUI _playerInventoryUI;
    public PlayerInventoryUI UI { get => _playerInventoryUI; }

    [SerializeField]
    private GiantInventoryUI _giantInventoryUI;

    [SerializeField]
    private GameObject _parcelPrefab;

    //database
    private DatabaseController databaseController;


    private static PlayerInventory instance;



    private void Awake()
    {
        instance = this;
        if (Gamemanager.Get == null) // 
        {
            return;
        }
        if (Gamemanager.Get.PlayerInventory == null)
        {
            Debug.Log("hai");
            Gamemanager.Get.PlayerInventory = instance;
        }
        else
        {
            Destroy(this.gameObject);
        }
        SceneSystem.Get.InventoryParent = this.gameObject;
    }

    /// <summary>
    /// Add a parcel to the inventory
    /// </summary>
    /// <param name="parcel"></param>
    /// <returns>
    /// <para>True when there is enough space to add a parcel </para>
    /// <para>False when there is no room in the inventory  </para>
    /// </returns>
    public bool AddParcel(Parcel parcel)
    {
        if (_parcelInventory.Count < _parcelInventorySize)
        {
            _parcelInventory.Add(parcel);
            parcel.transform.position = Vector3.zero;
            return true;
        }
        return false;   
    }

    public bool AddParcel(Rarity rarity)
    {
        if (_parcelInventory.Count < _parcelInventorySize)
        {
            Parcel newParcel = Instantiate(_parcelPrefab, new Vector3(0, 0, 0), Quaternion.identity).GetComponent<Parcel>();
            newParcel.rarity = rarity;
            newParcel.specialParcel = true;
            _parcelInventory.Add(newParcel);
            newParcel.gameObject.SetActive(false);
            return true;
        }
        return false;   
    }


    /// <summary>
    /// Used when depositing to the parcel giant
    /// </summary>
    /// <param name="parcel">Parcel gameobject deposited to the parcel giant</param>
    public void DepositParcel(Parcel parcel)
    {
        _parcelInventory.Remove(parcel);
        AddParcelDepositPointsToDatabase(parcel.rarity);
        Destroy(parcel.gameObject);
        UI.RefreshInventoryUI();
        _giantInventoryUI.RefreshGiantInventoryUI();
        if (PlayerVariables.dbConnectionFailed == false)
        {
            _giantInventoryUI.RefreshDatabaseInfo();
        }

        if (!PlayerVariables.playerParticipated)
        {
            Debug.Log("participated: " + PlayerVariables.playerParticipated + " Today: " + System.DateTime.Today + " Last Day participated: " + PlayerVariables.lastDayParticipated + "Online data last part: " + PlayerPrefs.GetString("date"));
            if (PlayerVariables.lastDayParticipated != System.DateTime.Today)
            {
                PlayerVariables.playerParticipated = true;
                PlayerVariables.lastDayParticipated = System.DateTime.Today;
                Debug.Log("Player part: " + PlayerVariables.playerParticipated);
                PlayerPrefs.SetString("date", System.DateTime.Today.ToString());
            }

        }
    }

    void OnGUI()
    {
        //Show participated on screen mobile debug
        //GUI.Label(new Rect(200, 400, 200, 40), "playerParticipated : " + PlayerVariables.playerParticipated);
    }

    public void DepositAllParcels()
    {
        List<Parcel> temp = new List<Parcel>();
        foreach (var parcel in _parcelInventory)
        {
            if (parcel.specialParcel)
            {
                temp.Add(parcel);
                AddParcelDepositPointsToDatabase(parcel.rarity);
            }
        }
        for (int i = 0; i < temp.Count; ++i)
        {
            Parcel toDestroy = temp[i];
            temp[i] = null;
            _parcelInventory.Remove(toDestroy);
            Destroy(toDestroy);
        }

        UI.RefreshInventoryUI();
        _giantInventoryUI.RefreshGiantInventoryUI();

        if (PlayerVariables.dbConnectionFailed == false)
        {
            _giantInventoryUI.RefreshDatabaseInfo();
        }

        if (!PlayerVariables.playerParticipated)
        {
            Debug.Log("opened legendary: " + PlayerVariables.playerParticipated + " Today: " + System.DateTime.Today + " Last Day participated: " + PlayerVariables.lastDayParticipated + "Online data last part: " + PlayerPrefs.GetString("date"));
            if (PlayerVariables.lastDayParticipated != System.DateTime.Today)
            {
                PlayerVariables.playerParticipated = false;
                PlayerVariables.lastDayParticipated = System.DateTime.Today;
                Debug.Log("Player part offline: " + PlayerVariables.playerParticipated);
                PlayerPrefs.SetString("date", System.DateTime.Today.ToString());
                PlayerPrefs.SetInt("playerPart", 0);
                Debug.Log("Player part database: " + PlayerPrefs.GetInt("playerPart"));
            }
        }
    }



    public void OpenNormalParcel(Parcel parcel)
    {
        _parcelInventory.Remove(parcel);
        Destroy(parcel.gameObject);
        UI.RefreshInventoryUI();
        _giantInventoryUI.RefreshGiantInventoryUI();

        // Add currency here
        if (PlayerVariables.dbConnectionFailed == false)
        {
            PlayerModifier moneyGained = new PlayerModifier();
            moneyGained.Money = +200;
            databaseController.PlayerModifierSubmit((moneyGained), player => OpenNormalParcelCallback(player));
        }
    }

    private void OpenNormalParcelCallback(PlayerInfo playerInfo)
    {
        _currency = playerInfo.Money;
        UI.RefreshCurrencyText();
    }

    public void GetPlayerCurrency()
    {
        if (PlayerVariables.dbConnectionFailed == false)
            databaseController.GetPlayerMoney((money) => GetPlayerCurrencyCallback(money));
    }

    private void GetPlayerCurrencyCallback(int fetchedCurrency)
    {
        _currency = fetchedCurrency;
        UI.RefreshCurrencyText();
    }


    //add points to the database when a parcel is deposited
    public void AddParcelDepositPointsToDatabase(Rarity rarity)
    {
        if (PlayerVariables.dbConnectionFailed == false)
        {
            Debug.Log("Database connection on. Add points from depositing a special parcel.");
            switch (rarity)
            {
                case Rarity.Common:
                    databaseController?.PointsSubmit((points) => RemoveOneParcelCallback(points), PlayerVariables.pointsCommonParcel, Rarity.Common);
                    Debug.Log($"added {PlayerVariables.pointsCommonParcel} points to the database");
                    break;
                case Rarity.Uncommon:
                    databaseController?.PointsSubmit((points) => RemoveOneParcelCallback(points), PlayerVariables.pointsUncommonParcel, Rarity.Uncommon);
                    Debug.Log($"added {PlayerVariables.pointsUncommonParcel} points to the database");
                    break;
                case Rarity.Rare:
                    databaseController?.PointsSubmit((points) => RemoveOneParcelCallback(points), PlayerVariables.pointsRareParcel, Rarity.Rare);
                    Debug.Log($"added {PlayerVariables.pointsRareParcel} points to the database");
                    break;
            }
        }
    }

    /// <summary>
    /// After a point is added to the database from parcel removal do this
    /// </summary>
    /// <param name="points"></param>
    private void RemoveOneParcelCallback(int points)
    {
        Debug.Log($"Total amount of points the player has {points}");
        UI.RefreshInventoryUI();
        _giantInventoryUI.RefreshDatabaseInfo();
    }

    private void PopulateInventory() // For testing
    {
        for (int i = 0; i < 11; ++i)
        {
            Parcel temp = Instantiate(_parcelPrefab, new Vector3(0, 0, 0), Quaternion.identity).GetComponent<Parcel>();
            temp.gameObject.SetActive(false);
            temp.gameObject.name = $"Parcel {i}";
            int rand = Random.Range(0, 4);
            if (rand < 3)
            {
                temp.rarity = (Rarity)Random.Range(0, 3);
                temp.specialParcel = true;
            }
            AddParcel(temp);
        }
    }


    private void Start()
    {
        //PopulateInventory();
        databaseController = GameObject.Find("Managers")?.GetComponent<DatabaseInitializer>()?.dbc;
        GetPlayerCurrency();
    }
}

{% endhighlight %}

</details>


<details>
<summary> Class to create a randomized stack of parcels based on a players inventory </summary>

{% highlight csharp %}
 
public class ParcelStack3D : MonoBehaviour
{

    [SerializeField]
    private List<GameObject> _parcelPrefabs = new List<GameObject>();
    /*
    *   Prefab list
    *   0 = Normal parcel
    *   1 = Uncommon parcel
    *   2 = Rare Parcel
    */

    [SerializeField]
    private List<Transform> _stackPositions = new List<Transform>();
    [SerializeField]
    private List<TextMeshPro> _playerNameText = new List<TextMeshPro>();

    private List<PlayerInfo> _playerList = new List<PlayerInfo>();


    [SerializeField]
    private float yOffset = 5;  // Y axis offset when spawning parcels

    [SerializeField]
    private float xOffset = 5;  // X axis offset

    private DatabaseController databaseController;

    void Awake()
    {
        databaseController = GameObject.Find("Managers")?.GetComponent<DatabaseInitializer>()?.dbc;
        //  StartCoroutine(PopulateStacks(_playerList));
    }

    private void OnEnable()
    {
        databaseController.GetAllPlayers((allPlayers) => OnEnableCallback(allPlayers));
        databaseController.GetPlayerInformation((exception, player) => OnEnableCallbackPlayer(player));
    }

    private void OnEnableCallback(List<PlayerInfo> allPlayers)
    {
        if (allPlayers == null) //no connection
        {
            Debug.Log("No Connection to db");
        }
        else //playerlist recieved from the database
        {
            _playerList = allPlayers;
            for (int i = 0; i < 3; ++i)
            {
                _playerNameText[i + 1].text = $"{i + 1}. {_playerList[i].Name}";
            }

        }
    }

    private void OnEnableCallbackPlayer(PlayerInfo player)
    {
        _playerNameText[0].text = player.Name;
        StartCoroutine(SpawnStack(player, _stackPositions[0]));
    }

    public void SpawnStack(int i)   // Parameter is what the number of player you want to spawn. 1 == Number 1 player, 2 Number 2 etx
    {
        StartCoroutine(SpawnStack(_playerList[i - 1], _stackPositions[i]));
    }

    public IEnumerator SpawnStack(PlayerInfo player, Transform spawnPos)
    {
        List<Rarity> rarities = new List<Rarity>();
        CreateParcelList(rarities, player);

        int iterations = 0;
        int xOffsetAmount = ((-rarities.Count - 1) / 5) + 2;

        foreach (var rarity in rarities)
        {
            Vector3 offsetSpawnPos = new Vector3(spawnPos.position.x + xOffset * xOffsetAmount,
                                           spawnPos.position.y + yOffset * iterations,
                                           spawnPos.position.z);
            switch (rarity)
            {
                case Rarity.Common:
                    Instantiate(_parcelPrefabs[0], offsetSpawnPos, Quaternion.identity, spawnPos);
                    break;
                case Rarity.Uncommon:
                    Instantiate(_parcelPrefabs[1], offsetSpawnPos, Quaternion.identity, spawnPos);
                    break;
                case Rarity.Rare:
                    Instantiate(_parcelPrefabs[2], offsetSpawnPos, Quaternion.identity, spawnPos);
                    break;
            }
            ++iterations;
            if (iterations == 5)
            {
                iterations = 0;
                ++xOffsetAmount;
            }
            yield return new WaitForSeconds(0.1f);
        }
    }

    private void CreateParcelList(List<Rarity> rarityList, PlayerInfo playerInfo)
    {
        int commonAmount = playerInfo.CommonParcels;
        int uncommonAmount = playerInfo.UncommonParcels;
        int rareAmount = playerInfo.RareParcels;

        for (int j = 0; j < commonAmount; j++)
        {
            rarityList.Add(Rarity.Common);
        }
        for (int j = 0; j < uncommonAmount; j++)
        {
            rarityList.Add(Rarity.Uncommon);
        }
        for (int j = 0; j < rareAmount; j++)
        {
            rarityList.Add(Rarity.Rare);
        }
        rarityList.Shuffle();
    }
}

static class ShuffleExtension   // Shuffle list with Fisher–Yates 
{
    public static void Shuffle<T>(this IList<T> list)
    {
        System.Random rng = new System.Random();

        int n = list.Count;
        while (n > 1)
        {
            n--;
            int k = rng.Next(n + 1);
            T value = list[k];
            list[k] = list[n];
            list[n] = value;
        }
    }
}

{% endhighlight %}

</details>



<details>
<summary> Class to handle clicking and touching different objects in the game </summary>

{% highlight csharp %}
 
public class ClickHandler : MonoBehaviour
{
    [SerializeField]
    private LayerMask _clickableLayerMask;
    public bool CanClick = true;
    [SerializeField]
    private Camera mainCam;

    private void Start()
    {
        Gamemanager.Get.clickHandler = this;
    }

#if UNITY_EDITOR_WIN
    void Update()
    {
        if (CanClick)
        {
            if (!EventSystem.current.IsPointerOverGameObject(-1))
            {
                if (Input.GetMouseButtonDown(0))
                {
                    Ray ray = mainCam.ScreenPointToRay(Input.mousePosition);
                    RaycastHit hit;
                    if (Physics.Raycast(ray, out hit, Mathf.Infinity, _clickableLayerMask))
                    {
                        hit.transform.GetComponent<IClickable>().OnClick();
                    }
                }
            }
        }
    }
#else
    void Update()
    {
        if (CanClick)
        {
            if (!EventSystem.current.IsPointerOverGameObject(0))
            {
                if (Input.touchCount > 0 && Input.GetTouch(0).phase == TouchPhase.Began)
                {
                    Ray ray = mainCam.ScreenPointToRay(Input.GetTouch(0).position);
                    RaycastHit hit;

                    if (Physics.Raycast(ray, out hit, Mathf.Infinity, _clickableLayerMask))
                    {
                        hit.transform.GetComponent<IClickable>().OnClick();
                    }
                }
            }
        }
    }

#endif

}

{% endhighlight %}

</details>



<details>
<summary> Adding a smooth snapping animation to Unitys scrollrect component </summary>

{% highlight csharp %}
 
public class SnapToValue : MonoBehaviour
{

    public IEnumerator SnapTo(ScrollRect scrollRect, float duration, int steps)
    {
        float time = 0;
        float endDragValue = scrollRect.horizontalScrollbar.value;
        float lerpEndValue;



        List<float> stepValues = new List<float>();
        int points = steps - 1;

        for (int i = 0; i < points; ++i)
        {
            stepValues.Add(i * (1 / (float)points));
        }
        stepValues.Add(1);

        foreach (var item in stepValues)
        {
            //Debug.Log(item);
        }



        float nearestStepValue = stepValues.OrderBy(x => Mathf.Abs(x - endDragValue)).First();
        lerpEndValue = nearestStepValue;

        /*
            if (endDragValue < 0.5)
            {
                lerpEndValue = 0;
            }
            else
            {
                lerpEndValue = 1;
            }
            */
        while (time < duration)
        {
            scrollRect.horizontalScrollbar.value = Mathf.Lerp(endDragValue, lerpEndValue, time / duration);
            time += Time.deltaTime;
            yield return null;
        }
    }
}

{% endhighlight %}

</details>



<details>
<summary> Scroll rect class itself to enable scrollbar snapping </summary>

{% highlight csharp %}
 
public class SnapScrollRect : ScrollRect
{

    public bool _snapScrollbar;
    [SerializeField]
    private SnapToValue _snap;
    [SerializeField]
    private int _steps;

    public override void OnEndDrag(PointerEventData eventData)
    {
        if (_snapScrollbar)
            StartCoroutine(_snap.SnapTo(this, 0.2f, _steps));

    }

}

{% endhighlight %}

</details>
