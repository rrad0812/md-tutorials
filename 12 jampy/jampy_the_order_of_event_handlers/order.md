
# Events on Jam.py framework fields

## Main path

The main path is the directory `jam/js/modules`, and almost all field changes are merged into `Field.set_value()`.

The most important thing is to follow three points:

- event declarations:events.py:1-112
- event form processing:abstr_item.js:699-749
- changing field values:field.js:327-423

### Event handlers of a field being edited

```txt
input blur/change
  -> DBAbstractInput.change_field_text()
  -> field.text / field.value
  -> Field.set_value()
  -> item.edit()                 // if record is not in edit state
  -> on_before_edit
  -> on_after_edit
  -> on_before_field_changed
  -> save field.data
  -> update lookup/slave field
  -> tag item as modified
  -> on_field_changed
  -> update_controls()
```

When entered, the start of the path is at `input.js:620-675`, and the center point is `Field.set_value()` at `field.js:366-423`.

For the fastest tracking, set breakpoints here:

```js
// field.js
Field.set_value()
Field._do_before_changed()
Field._do_after_changed()

// item.js
Item._edit()
Item._append()
Item._process_apply()
Item._process_event()
```

Pay special attention to `_process_event()` u `abstr_item.js:699-749`.

### Event hendlers of a form

For form events, the order is generally:

```js
task.on_edit_form_*
  -> owner.on_edit_form_*
  -> item.on_edit_form_*
```

For `close_query`, `keyup` and `keydown` the order is different: first the `item` is checked, then the `owner`, then the `task`.

### Console.trace

You can also temporarily add `console.trace()` in `field.js`:

```js
_do_before_changed() {
    console.log('on_before_field_changed', this.field_name);
    console.trace();

    if (this._owner_is_item()) {
        if (!this.owner.is_changing()) {
            throw new Error(task.language.not_edit_insert_state.replace('%s', this.owner.item_name));
        }
        if (this.owner.on_before_field_changed) {
            this.owner.on_before_field_changed.call(this.owner, this);
        }
    }
}

_do_after_changed(lookup_item) {
    console.log('on_field_changed', this.field_name);
    console.trace();

    if (this.owner && this.owner.on_field_changed) {
        this.owner.on_field_changed.call(this.owner, this, lookup_item);
    }
    // ...
}
```

### Event hendlers of a lookup field

For the lookup field, the path is:

```txt
select_value()
  -> _do_select_value()
  -> on_field_select_value
  -> open lookup form
  -> choose records
  -> Field.set_value(...)
  -> on_before_field_changed
  -> on_field_changed
```

`_do_select_value()` is located at the end `field.js:1123-1155`.

Practically, for one field I would first put a breakpoint in:

```js
Field.set_value()
Field._change_lookup_field()
Field._do_before_changed()
Field._do_after_changed()
```

This is where you'll catch almost all the actual changes, including `checkbox`, `lookup`, and `typeahead`. `events.py` serves as a map of the available `on_...` names, but the actual order is determined by the calls to `item.js`, `input.js` and `field.js`.
