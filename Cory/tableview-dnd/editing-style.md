1 - **Set the table to editing (I do it in viewWillAppear)**

```
override func viewWillAppear(_ animated: Bool) {
   super.viewWillAppear(animated: animated)
   
   tableView.setEditing(true, animated: false)
}
```

2 - **Hide the default accessory icon:**

```
override func tableView(_ tableView: UITableView, editingStyleForRowAt indexPath: IndexPath) -> UITableViewCellEditingStyle {
    return .none
}
```

3 - **Keep editing mode from moving everything to the right (in the cell)**

```
override func tableView(_ tableView: UITableView, shouldIndentWhileEditingRowAt indexPath: IndexPath) -> Bool {
    return false
}
```

4 - Not required in swift 3

5 - Reorder your array

**Extra Note**

If you want to have your table cells selectable, add the following code within `viewWillAppear()` function.

```
tableView.allowsSelectionDuringEditing = true
```

