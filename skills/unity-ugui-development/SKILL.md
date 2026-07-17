---
name: unity-ugui-development
description: Conventions for Unity UGUI development. ALWAYS Auto-loaded when writing UI code for Unity Projects using UGUI.
---

# Unity UGUI Development

Conventions for developing UI with Unity's UGUI system.

## Button Event Handling

**Do NOT use Unity UGUI's button OnClick configuration in the inspector.**

Instead, reference the button in a script field and subscribe to its OnClick event in code:

```csharp
public class MyUIController : MonoBehaviour
{
    [SerializeField] private Button _submitButton;

    private void Start()
    {
        _submitButton.onClick.AddListener(HandleSubmitClicked);
    }

    private void HandleSubmitClicked()
    {
        // Handle click
    }
    
    private void OnDestroy()
    {
        _submitButton.onClick.RemoveListener(HandleSubmitClicked);
    }
}
```

**Reason:** Inspector-configured button events are easy to accidentally break and harder to track in code.

## Class Naming Conventions

| Suffix | Purpose | Example |
|--------|---------|---------|
| UIPanel | Container | SettingsUIPanel, InventoryUIPanel |
| UIComponent | Container with specific functionality | ListUIComponent, TabUIComponent |
| UIItem | Item that is part of a list | PlayerListUIItem, InventorySlotUIItem |

## Property-Based Data Binding

Bind uGUI text elements through properties for clean data access:

```csharp
[SerializeField] private TMP_Text _titleText;

public string Title
{
    get => _titleText.text;
    set => _titleText.text = value;
}
```

## Dependency Inversion via Delegates

Smaller components should NOT reference their containing elements. Use events/delegates to communicate upward:

```csharp
// UIItem does NOT know about its container
public class PlayerListUIItem : MonoBehaviour
{
    public event Action<PlayerListUIItem> OnSelected;
    public event Action<PlayerListUIItem> OnDeleteRequested;

    [SerializeField] private Button _selectButton;
    [SerializeField] private Button _deleteButton;

    private void Awake()
    {
        _selectButton.onClick.AddListener(() => OnSelected?.Invoke(this));
        _deleteButton.onClick.AddListener(() => OnDeleteRequested?.Invoke(this));
    }
}

// Container subscribes to item events
public class PlayerListUIComponent : MonoBehaviour
{
    private void SetupItem(PlayerListUIItem item)
    {
        item.OnSelected += HandleItemSelected;
        item.OnDeleteRequested += HandleItemDeleteRequested;
    }
}
```

## Memento Pattern (State Persistence)

When external access to UI state is needed, components provide methods to load and export their state:

```csharp
public class CharacterCreationUIPanel : MonoBehaviour
{
    // Load state into UI
    public void SetData(CharacterData data)
    {
        Name = data.Name;
        Class = data.Class;
        Level = data.Level;
    }

    // Export current UI state
    public CharacterData ExportData()
    {
        return new CharacterData
        {
            Name = Name,
            Class = Class,
            Level = Level
        };
    }

    // Refresh UI from stored data source
    public void RefreshFromDataSource()
    {
        SetData(_currentDataSource);
    }
}
```
