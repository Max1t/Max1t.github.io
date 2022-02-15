# DodoFarm

Student project. A working proof of concept demo was created with proper models and sounds.

![Image](/assets/dodomenu.png)


## Features

- AI driven dodos that you have to try and keep alive from almost everything
- Dodos are generated and feature different colors and when you breed more dodos you can aqquire more
- Multiplayer where 2 people can try and save the dodos at the same time.  Dynamic splitscreen effect. seamlessly move between shared and separate view
- Different hazards that you need to watch out. Hazards have sound and visual indicators on the ground and affect the dodos in different ways
- AI raptors that try to snatch away dodos. Dodos need to be kept safe during these swarms

![Image](/assets/dodomp.png)

Example of the multiplayer splitscreen effect.

## Some Code snippets

Full source is in a private repository.
Here are some code snippets from various things I worked on.


<details>
<summary> Control handler for switching between keyboard and gamepad controls</summary>

{% highlight csharp %}
 

public class ControlHandler : MonoBehaviour
{


    public delegate void OnControlChanged();
    public OnControlChanged OnControlsChangedCallback;

    public KeyboardControl _keyboardControl;
    public GamepadControl _gamepadControl;

    [SerializeField] private bool _keyboardOn;
    [SerializeField] private bool _gamepadOn;
    private bool _touchHeld = false;
    public bool TouchHeld { get => _touchHeld; set => _touchHeld = value; }

    public Camera _camera;



    public bool CurrentControlTouchHeld()
    {
        if (_keyboardOn) return _keyboardControl.HoldingTouch;
        if (_gamepadOn) return _gamepadControl.HoldingTouch;
        return false;
    }

    public bool CurrentControlState()
    {
        if (_keyboardOn) return _keyboardControl.On;
        if (_gamepadOn) return _gamepadControl.On;
        return false;
    }

    public bool KeyboardOn
    {
        get => _keyboardOn;
        set
        {
            _keyboardOn = value;
            if (OnControlsChangedCallback != null) OnControlsChangedCallback.Invoke();
        }
    }

    public bool GamepadOn
    {
        get => _gamepadOn;
        set
        {
            _gamepadOn = value;
            if (OnControlsChangedCallback != null) OnControlsChangedCallback.Invoke();
        }
    }


    public void updateControls()
    {
        if (KeyboardOn && !_keyboardControl.enabled) _keyboardControl.enabled = true;
        else if (!KeyboardOn && _keyboardControl.enabled) _keyboardControl.enabled = false;

        if (GamepadOn && !_gamepadControl.enabled) _gamepadControl.enabled = true;
        else if (!GamepadOn && _gamepadControl.enabled) _gamepadControl.enabled = false;
    }


    public void EnableCurrentControls(InputUser user)
    {
        if (_keyboardOn) _keyboardControl.EnableControlsUser(user);
        if (_gamepadOn) _gamepadControl.EnableControlsUser(user);
    }


    public void DisableCurrentControls(InputUser user)
    {
        if (_keyboardOn) _keyboardControl.DisableControlsUser(user);
        if (_gamepadOn) _gamepadControl.DisableControlsUser(user);
    }

    void OnEnable()
    {
        OnControlsChangedCallback += updateControls;
        if (OnControlsChangedCallback != null) OnControlsChangedCallback.Invoke();
    }

    void OnDisable()
    {
        OnControlsChangedCallback -= updateControls;
    }

    public void toggleKeyboard()
    {
        if (KeyboardOn) KeyboardOn = false;
        else KeyboardOn = true;
    }

    public void toggleGamepad()
    {
        if (GamepadOn) GamepadOn = false;
        else GamepadOn = true;
    }

    public void CreateControls(InputDevice device, InputUser user, bool tkp)
    {
        switch (device.description.deviceClass)
        {
            case (""):
                KeyboardOn = false;
                GamepadOn = true;
                _gamepadControl.InitializeControlsForPlayer(user);
                break;
            case ("Keyboard"):
                GamepadOn = false;
                KeyboardOn = true;
                if (tkp)
                {
                    _keyboardControl.InitializeControlsForPlayer2(user);
                    break;
                }
                _keyboardControl.InitializeControlsForPlayer(user);
                break;
        }
    }

{% endhighlight %}

</details>

<details>
<summary> Camera controller </summary>

{% highlight csharp %}
 
public class CameraMovement : MonoBehaviour
{

    public Transform _target = null;
    private Vector3 _targetPos;
    [SerializeField] private float smoothing = 2f;
    private Vector3 nullVelocity = Vector3.zero;
    private Vector3 _offset;
    [SerializeField] Camera attachedCamera;

    [Range(1, 10)]
    public float _cameraZ;
    [Range(1, 10)]
    public float _cameraY;
    [Range(-10, 10)]
    public float _cameraX;
    [Range(0, 90)]
    public float _cameraRot;

    public float _spcXOffset = 0; // Splitscreen offset
    public float _spcYOffset = 0; // Splitscreen offset
    public float _spcZOffset = 0; // Splitscreen offset

    public Vector3 _relativePos;

    public bool _Ready = false;

    private void OnValidate()
    {
        if (_target)
        {
            SetPos();
            calcOffset();
        }
    }

    public void calcOffset()
    {
        _offset = _targetPos - _target.position;
    }

    public void SetPos()
    {
        transform.position = new Vector3(_target.position.x + _cameraX,
                                 _target.position.y + _cameraY,
                                 _target.position.z + _cameraZ);
        transform.rotation = Quaternion.identity;
        transform.Rotate(_cameraRot, 180, 0);
    }

    void Update()
    {
        if (_target)
        {
            _relativePos = _target.position + _offset;
            Vector3 targetPosition = _target.position + _offset;
            if (_spcXOffset != 0) targetPosition += new Vector3(_spcXOffset, _spcYOffset, 0);

            transform.rotation = Quaternion.Lerp(transform.rotation, Quaternion.Euler(_cameraRot + _spcZOffset, 180, 0), 0.1f);
            transform.position = Vector3.SmoothDamp(transform.position, targetPosition, ref nullVelocity, smoothing);
        }
    }

    public void TargetObject(Transform target, bool rotate)
    {
        //_Ready = false;
        _target = target;
        _targetPos = new Vector3(target.position.x + _cameraX,
                                         target.position.y + _cameraY,
                                         target.position.z + _cameraZ);
        if (rotate)
        {
            transform.rotation = Quaternion.identity;
            transform.Rotate(_cameraRot, 180, 0);
        }
        calcOffset();
        //if (gameObject.activeSelf) StartCoroutine(MoveToTarget(_target.position + _offset));
        //else
        //{
        //    SetPos();
        //    _Ready = true;sa
        //}

    }

    IEnumerator MoveToTarget(Vector3 targetPosition)
    {
        while (!_Ready)
        {
            transform.position = Vector3.SmoothDamp(transform.position, targetPosition, ref nullVelocity, smoothing);
            if (transform.position == targetPosition) _Ready = true;
            yield return null;
        }
    }
}

{% endhighlight %}

</details>

<details>
<summary> Controller class for the dynamic splitscreen effect. First time ever attempting this so implementation may need some work </summary>

{% highlight csharp %}
 
public class SplitscreenControl : MonoBehaviour
{
    [SerializeField] Camera _SingleCamera;
    Camera cam1;
    Camera cam2;

    [SerializeField] GameObject _PlayerOne;
    [SerializeField] GameObject _PlayerTwo;

    [SerializeField] GameObject _PlayerOneCone;
    [SerializeField] GameObject _PlayerTwoCone;

    [SerializeField] float _spcCutoff;
    [SerializeField] float _spcXOffset;
    [SerializeField] float _spcZOffset;
    [SerializeField] float _spcYOffset;


    [SerializeField] CameraMovement _PlayerOneCameraScript;
    [SerializeField] CameraMovement _PlayerTwoCameraScript;


    [SerializeField] Camera _MaskCamera;

    [SerializeField] RenderTexture _MaskCameraRenderTexture;

    [SerializeField] RenderTexture _PlayerTwoCameraRenderTexture;

    [SerializeField] PostProcessVolume _volume;

    [SerializeField] LayerMask _mask;

    public GameObject _midPoint;
    bool _setup = false;

    SplitScreen _spcSettings;
    [SerializeField] private bool _split = false;

    public bool Setup { get => _setup; set => _setup = value; }

    public void SetupPlayers(List<GameObject> playerList)
    {
        _PlayerOne = playerList[0];
        _PlayerTwo = playerList[1];

        _PlayerOneCone = Instantiate(_PlayerOneCone, _PlayerOne.transform.position, Quaternion.identity);
        _PlayerTwoCone = Instantiate(_PlayerTwoCone, _PlayerTwo.transform.position, Quaternion.identity);

        Setup = true;
    }

    public void SetupMainCamera(List<Camera> camList)
    {

        var layer = camList[1].gameObject.GetComponent<PostProcessLayer>();
        layer.volumeLayer.value = _mask;

        _volume.profile.TryGetSettings(out _spcSettings); // Get post processing settings

        _spcSettings.enabled.value = true; // Set splitscreen post processing to enabled

        _spcSettings._MaskTex.value = _MaskCameraRenderTexture; // Set Splitscreen effects mask to target texture

        camList[1].targetTexture = new RenderTexture(Screen.width, Screen.height, 24); // New render texture

        _spcSettings._Tex.value = camList[1].targetTexture; // Set second players camera texture to shaders Second camera texture

        _PlayerOneCameraScript = camList[0].gameObject.GetComponent<CameraMovement>();
        _PlayerTwoCameraScript = camList[1].gameObject.GetComponent<CameraMovement>();

        cam1 = camList[0];
        cam2 = camList[1];
    }

    public void ResetSplitscreenControl()
    {
        _volume.profile.TryGetSettings(out _spcSettings); // Get post processing settings

        _spcSettings.enabled.value = false; // Set splitscreen post processing to enabled

        _spcSettings._MaskTex.value = null; // Set Splitscreen effects mask to target texture

        _spcSettings._Tex.value = null; // Set second players camera texture to shaders Second camera texture

        _PlayerOneCameraScript = null;
        _PlayerTwoCameraScript = null;

        cam1 = null;
        cam2 = null;

        _setup = false;
    }

    void Update()
    {
        if (Setup)
        {
            _PlayerOneCone.transform.position = _PlayerOne.transform.position;
            _PlayerTwoCone.transform.position = _PlayerTwo.transform.position;


            Vector3 midPointXZ =
                    new Vector3((_PlayerOne.transform.position.x +
                                (_PlayerTwo.transform.position.x - _PlayerOne.transform.position.x) / 2),
                                _MaskCamera.gameObject.transform.position.y,
                                _PlayerOne.transform.position.z +
                                (_PlayerTwo.transform.position.z - _PlayerOne.transform.position.z) / 2);

            Vector3 midPoint =
                                        new Vector3((_PlayerOne.transform.position.x +
                                (_PlayerTwo.transform.position.x - _PlayerOne.transform.position.x) / 2),
                                (_PlayerOne.transform.position.y +
                                (_PlayerTwo.transform.position.y - _PlayerOne.transform.position.y) / 2),
                                _PlayerOne.transform.position.z +
                                (_PlayerTwo.transform.position.z - _PlayerOne.transform.position.z) / 2);


            if (Vector3.Distance(_PlayerOne.transform.position, _midPoint.transform.position) < _spcCutoff && _split)
            {
                // cam1.enabled = false;
                // cam2.enabled = false;
                //_SingleCamera.enabled = true;
                //_spcSettings.enabled.value = false;
                _PlayerOneCameraScript.TargetObject(_midPoint.transform, false);
                _PlayerTwoCameraScript.TargetObject(_midPoint.transform, false);
                _split = false;
            }
            else if (Vector3.Distance(_PlayerOne.transform.position, _midPoint.transform.position) > _spcCutoff && !_split)
            {
                cam1.enabled = true;
                cam2.enabled = true;
                //_SingleCamera.enabled = false;
                _spcSettings.enabled.value = true;
                _PlayerOneCameraScript.TargetObject(_PlayerOne.transform, false);
                _PlayerTwoCameraScript.TargetObject(_PlayerTwo.transform, false);
                _split = true;
            }

            _MaskCamera.gameObject.transform.position = midPointXZ;


            _midPoint.transform.position = midPoint;


            var z1 = _PlayerOne.transform.position.z - midPoint.z;
            var z2 = _PlayerTwo.transform.position.z - midPoint.z;

            if (z1 < 0) z1 = -z1;
            if (z2 < 0) z2 = -z2;

            if (_split)
            {
                // X Camera offset
                // When a player is on the right camera moves slighly left
                // Vice versa if player is on the left.
                if (_PlayerOne.transform.position.x > _PlayerTwo.transform.position.x)   // Player 1 Left
                {                                                                        // Player 2 Right
                    _PlayerOneCameraScript._spcXOffset =
                    Mathf.Lerp(0,
                               -_spcXOffset,
                               (Mathf.Clamp(_PlayerOne.transform.position.x - _PlayerTwo.transform.position.x, 0, 10) / 10));

                    _PlayerTwoCameraScript._spcXOffset =
                    Mathf.Lerp(0,
                               _spcXOffset,
                               (Mathf.Clamp(_PlayerOne.transform.position.x - _PlayerTwo.transform.position.x, 0, 10) / 10));
                }
                else                                                                    //  Player 1 Right         
                {                                                                       //  Player 2 Left 
                    _PlayerOneCameraScript._spcXOffset =
                    Mathf.Lerp(0,
                               _spcXOffset,
                               (Mathf.Clamp(_PlayerTwo.transform.position.x - _PlayerOne.transform.position.x, 0, 10) / 10));

                    _PlayerTwoCameraScript._spcXOffset =
                    Mathf.Lerp(0,
                               -_spcXOffset,
                               (Mathf.Clamp(_PlayerTwo.transform.position.x - _PlayerOne.transform.position.x, 0, 10) / 10));
                }

                //-------------------------------------------------------------------------//
                //-------------------------------------------------------------------------// 
                //-------------------------------------------------------------------------//

                if (_PlayerOne.transform.position.z > _PlayerTwo.transform.position.z)  //Player 1 is above Player 2
                {
                    _PlayerOneCameraScript._spcZOffset =
                    Mathf.Lerp(0,
                               -_spcZOffset,
                               (Mathf.Clamp(_PlayerOne.transform.position.z - _PlayerTwo.transform.position.z, 0, 10) / 10));


                    _PlayerTwoCameraScript._spcZOffset =
                    Mathf.Lerp(0,
                               _spcZOffset,
                               (Mathf.Clamp(_PlayerOne.transform.position.z - _PlayerTwo.transform.position.z, 0, 10) / 5));
                }
                else
                {
                    _PlayerOneCameraScript._spcZOffset =
                    Mathf.Lerp(0,
                               _spcZOffset,
                               (Mathf.Clamp(_PlayerTwo.transform.position.z - _PlayerOne.transform.position.z, 0, 10) / 10));

                    _PlayerTwoCameraScript._spcZOffset =
                    Mathf.Lerp(0,
                               -_spcZOffset,
                               (Mathf.Clamp(_PlayerTwo.transform.position.z - _PlayerOne.transform.position.z, 0, 10) / 10));
                }
            }
            else
            {
                _PlayerTwoCameraScript._spcXOffset = 0;
                _PlayerOneCameraScript._spcXOffset = 0;
                _PlayerTwoCameraScript._spcZOffset = 0;
                _PlayerOneCameraScript._spcZOffset = 0;
            }

            if (cam1.transform.position.x == midPoint.x)
            {
                _spcSettings.enabled.value = false;
                //_split = false;
            }
        }
    }
}

{% endhighlight %}

</details>

<details>
<summary> Audio manager class using FMOD studio </summary>

{% highlight csharp %}
 
public class AudioManager : MonoBehaviour
{
    public static AudioManager manager;
    public AudioData _data;

    EventInstance _atmoInstance;
    EventInstance _musicInstance;

    [Range(0, 1)]
    [SerializeField]
    private float _masterVolume = 1f;
    Bus masterBus;

    private void Awake()
    {
        if (manager == null)
        {
            manager = this;
            DontDestroyOnLoad(manager);
        }
        else if (manager != this)
        {
            Destroy(this);
        }

        StartMusic();
    }

    void Start()
    {
        masterBus = RuntimeManager.GetBus("Bus:/");
        _masterVolume = Settings.MasterVolume;
        masterBus.setVolume(_masterVolume);
    }

    private void OnValidate()
    {
        masterBus.setVolume(_masterVolume);
    }

    public void OnValueChanged(Slider slider)
    {
        _masterVolume = slider.value;
        masterBus.setVolume(_masterVolume);
    }

    /*private void OnEnable()
    {
        StartAtmo();
        StartMusic();
    }*/

    public void OnStartGame()
    {
        StartAtmo();
    }

    public void OnStopGame()
    {
        StopAtmo();
    }

    private void StopAllSounds()
    {
        masterBus.stopAllEvents(FMOD.Studio.STOP_MODE.IMMEDIATE);
    }

    private void StopAtmo()
    {
        _atmoInstance.stop(FMOD.Studio.STOP_MODE.IMMEDIATE);
        _atmoInstance.release();
    }

    public void PauseAtmoSmooth()
    {
        _atmoInstance.stop(FMOD.Studio.STOP_MODE.ALLOWFADEOUT);
    }

    public void ResumeAtmo()
    {
        _atmoInstance.start();
    }

    public void StartAtmo()
    {
        _atmoInstance = RuntimeManager.CreateInstance(_data.Atmosphere.Atmo);
        _atmoInstance.start();
    }

    public void StartMusic()
    {
        _musicInstance = RuntimeManager.CreateInstance(_data.Music.Music);
        _musicInstance.start();
    }

    /// <summary>
    /// changes music.
    /// </summary>
    /// <param name="val">0 = normal music, 2 = raptor rush music</param>
    public void ChangeMusic(float val)
    {
        _musicInstance.setParameterByName("MUSIC_TRANS", val);
    }

    public void StopMusic()
    {
        _musicInstance.stop(FMOD.Studio.STOP_MODE.IMMEDIATE);
        _musicInstance.release();
    }

    public void PlayOneShot(string audio, Vector3 position)
    {
        RuntimeManager.PlayOneShot(audio, position);
    }

    /// <summary>
    /// Creates new instance and starts it immediately
    /// </summary>
    /// <param name="audio"></param>
    /// <param name="position"></param>
    /// <returns></returns>
    public EventInstance NewLoopSound(string audio, Vector3 position)
    {
        EventInstance audioInstance = RuntimeManager.CreateInstance(audio);
        audioInstance.start();
        return audioInstance;
    }

    public void StartSound(EventInstance instance)
    {
        instance.start();
    }

    public void PauseSound(EventInstance instance)
    {
        instance.stop(FMOD.Studio.STOP_MODE.IMMEDIATE);
    }

    public void StopSound(EventInstance instance)
    {
        instance.stop(FMOD.Studio.STOP_MODE.IMMEDIATE);
        instance.release();
    }
}

{% endhighlight %}

</details>

<details>
<summary> Generic class to pool any monobehavior classes </summary>

{% highlight csharp %}
 
public abstract class GenericPool<T> : MonoBehaviour where T : Component
{

    [SerializeField] private T prefab;
    [SerializeField] private List<T> _objectPool = new List<T>();

    public static GenericPool<T> Instance;
    private void Awake()
    {
        Instance = this;
    }

    public T Get()
    {
        foreach (var item in _objectPool)
        {
            if (!item.gameObject.activeSelf)
            {
                return item;
            }
        }
        return AddAndReturn();
    }

    private T AddAndReturn()
    {
        T newObject = Instantiate(prefab);
        newObject.gameObject.SetActive(false);
        _objectPool.Add(newObject);
        newObject.transform.parent = gameObject.transform;
        return newObject;
    }

    public void Put(T objectToReturn)
    {
        objectToReturn.gameObject.SetActive(false);
    }

    private void Add(int x)
    {
        for (int i = 0; i < x; i++)
        {
            T newObject = Instantiate(prefab);
            newObject.gameObject.SetActive(false);
            _objectPool.Add(newObject);
            newObject.transform.parent = gameObject.transform;
        }
    }


}

{% endhighlight %}

</details>
