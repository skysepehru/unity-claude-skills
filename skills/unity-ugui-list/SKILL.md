---
name: unity-ugui-list
description: Generate uGUI list components where elements are created from a list of data.
---

# Unity uGUI List Pattern

Generate list-type UI components using the ListUIComponent/ListUIComponentItem pattern.

## Overview

This pattern provides:
- Type-safe generic list containers
- Automatic item instantiation from data
- Selection management (optional)
- Data export/import (Memento pattern)
- Item reordering

## Base Classes

### ListUIComponent (Container)

```csharp
public abstract class ListUIComponent<TListUIComponentItem, TListUIComponentItemDataType> : MonoBehaviour
    where TListUIComponentItem : ListUIComponentItem<TListUIComponentItem, TListUIComponentItemDataType>
    where TListUIComponentItemDataType : class, ICloneable<TListUIComponentItemDataType>, IEquatable<TListUIComponentItemDataType>
{
    [Inject] private DiContainer _diContainer;
    [SerializeField] private Transform _itemParent;
    [SerializeField] private TListUIComponentItem _itemPrefab;

    private readonly List<TListUIComponentItem> _items = new();

    public int CurrentSelectedItemIndex { get; private set; } = -1;
    public bool IsItemSelected => IsSelectable && CurrentSelectedItemIndex != -1;
    public bool IsEmpty => _items.Count == 0;
    public TListUIComponentItemDataType CurrentSelectedItemData =>
        IsItemSelected ? _items[CurrentSelectedItemIndex].CurrentDataSource : null;

    public event Action<TListUIComponentItemDataType> OnItemSelected;
    public event Action<TListUIComponentItemDataType> OnItemsUnselected;

    // Override to enable/disable selection behavior
    protected abstract bool IsSelectable { get; }

    // Load items from data list
    public void LoadItemsFromDataSourceList(List<TListUIComponentItemDataType> dataSourceList);

    // Export current items as data list
    public List<TListUIComponentItemDataType> ExportDataSourceListFromItems();

    // Item management
    public void AddItem(TListUIComponentItemDataType data, bool useClone = false);
    public void RemoveItemAtIndex(int index);
    public void RemoveItemByDataSource(TListUIComponentItemDataType data);
    public void RemoveCurrentSelectedItem();
    public void ClearItems();

    // Reordering
    public void MoveItemAtIndexUp(int index);
    public void MoveItemAtIndexDown(int index);
    public void MoveCurrentSelectedItemUp();
    public void MoveCurrentSelectedItemDown();

    // Modification
    public void ModifySelectedItem(Action<TListUIComponentItemDataType> modifier);
    public void ModifyItemAtIndex(int index, Action<TListUIComponentItemDataType> modifier);
    public void UpdateAllItemsWithTheirDataSource();
}
```

### ListUIComponentItem (Item)

```csharp
public abstract class ListUIComponentItem<TItemType, TItemDataType> : MonoBehaviour
    where TItemType : ListUIComponentItem<TItemType, TItemDataType>
{
    [SerializeField] private Button _button;
    [SerializeField] private GameObject _selectedIndicator;

    public event Action<TItemType> OnSelected;
    public event Action<TItemType> OnRequestDelete;

    public TItemDataType CurrentDataSource { get; private set; }

    public bool IsSelected
    {
        get => _selectedIndicator.activeSelf;
        set => _selectedIndicator.SetActive(value);
    }

    protected virtual void Awake()
    {
        _button.onClick.AddListener(() => OnSelected?.Invoke((TItemType)this));
        _selectedIndicator.SetActive(false);
    }

    public void SetData(TItemDataType data)
    {
        CurrentDataSource = data;
        ShowUIWithData(data);
    }

    // Implement to display data in UI elements
    protected abstract void ShowUIWithData(TItemDataType data);

    public void RefreshUIWithDataSource() => ShowUIWithData(CurrentDataSource);

    public void RequestDelete() => OnRequestDelete?.Invoke((TItemType)this);
}
```

## Data Interface Requirements

Data types must implement:

```csharp
public interface ICloneable<T>
{
    T Clone();
}

// Use System.IEquatable<T>
```

## Usage Example

### 1. Define Data Model

```csharp
public class PlayerData : ICloneable<PlayerData>, IEquatable<PlayerData>
{
    public string Id { get; set; }
    public string Name { get; set; }
    public int Score { get; set; }

    public PlayerData Clone() => new PlayerData { Id = Id, Name = Name, Score = Score };
    public bool Equals(PlayerData other) => other != null && Id == other.Id;
}
```

### 2. Create Item Class

```csharp
public class PlayerListUIItem : ListUIComponentItem<PlayerListUIItem, PlayerData>
{
    [SerializeField] private TMP_Text _nameText;
    [SerializeField] private TMP_Text _scoreText;

    public string Name
    {
        get => _nameText.text;
        set => _nameText.text = value;
    }

    public int Score
    {
        get => int.Parse(_scoreText.text);
        set => _scoreText.text = value.ToString();
    }

    protected override void ShowUIWithData(PlayerData data)
    {
        Name = data.Name;
        Score = data.Score;
    }
}
```

### 3. Create Component Class

```csharp
public class PlayerListUIComponent : ListUIComponent<PlayerListUIItem, PlayerData>
{
    // Set to true for selectable list, false for click-to-action list
    protected override bool IsSelectable => true;
}
```

### 4. Use in Panel

```csharp
public class LeaderboardUIPanel : MonoBehaviour
{
    [SerializeField] private PlayerListUIComponent _playerList;

    private void Start()
    {
        _playerList.OnItemSelected += HandlePlayerSelected;
    }

    public void LoadPlayers(List<PlayerData> players)
    {
        _playerList.LoadItemsFromDataSourceList(players);
    }

    private void HandlePlayerSelected(PlayerData player)
    {
        Debug.Log($"Selected: {player.Name}");
    }
}
```

## Folder Placement

Always create list UI files as a **pair** in a **named subfolder** under the feature's `UI/` folder:

```
Scripts/{Feature}/UI/{ListName}/
├── {ListName}UIComponent.cs    # The list container (manages the collection)
└── {ListName}UIItem.cs         # Individual list item (one per row)
```

Example:
```
Scripts/Exercises/UI/ExerciseList/
├── ExerciseListUIComponent.cs
└── ExerciseListUIItem.cs
```

## Inspector Setup

1. **Item Prefab**: Create prefab with ListUIComponentItem subclass
   - Add Button component (for selection)
   - Add selected indicator GameObject (hidden by default)
   - Add UI elements for data display

2. **Component**: Add ListUIComponent subclass to parent
   - Assign item prefab reference
   - Assign item parent transform (content container)

## Selection Behavior

- `IsSelectable = true`: Visual selection, deselect previous, track index
- `IsSelectable = false`: Click fires event but no visual state management
