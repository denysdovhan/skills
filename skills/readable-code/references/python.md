# Python Rules

Language-specific rules for Python. Apply them on top of the general rules in
[SKILL.md](../SKILL.md). Rules numbered from 15 to continue the general list.

## 15. Prefer Comprehensions — But Keep Them Simple

- Use list/dict/set comprehensions for **simple transforms and filters**.
- If a comprehension needs a nested loop AND a conditional, it's too complex —
  use a regular loop or extract a helper.
- Generator expressions for large sequences to avoid materializing the whole
  list.

```python
# Good: simple and readable
names = [user.name for user in users if user.is_active]

# Bad: too much going on
result = [transform(item) for group in data for item in group.items if item.valid and item.type == "A"]

# Good: break it up
valid_items = get_valid_items(data, item_type="A")
result = [transform(item) for item in valid_items]
```

## 16. Use Unpacking

- **Tuple unpacking** over index access: `name, age = get_user()` not
  `result[0], result[1]`.
- **Star unpacking** for head/tail: `first, *rest = items`.
- **Dict unpacking** with `**` for merging dicts.
- Unpacking makes the structure of the data explicit in the code.

## 17. Use `enumerate`, `zip`, and Itertools

- Use `enumerate(items)` — never track indices manually with `i += 1`.
- Use `zip(a, b)` to iterate in parallel — never index into parallel lists.
- Use `itertools` (`chain`, `groupby`, `islice`) before writing manual
  iteration logic.

## 18. Use Dataclasses and NamedTuples Over Raw Dicts/Tuples

- If a dict always has the same keys, it should be a `dataclass` or
  `NamedTuple`.
- If a function returns more than 2 values, return a `dataclass` or
  `NamedTuple` — not a raw tuple.
- This gives you names, type hints, and readable attribute access for free.

## 19. Use `pathlib` for File Paths

- Use `pathlib.Path` instead of `os.path.join` and string manipulation.
- `Path` objects are readable, composable (`/` operator), and cross-platform.

## 20. Design for `black` / `ruff format`

The general auto-formatting rule applies. In Python, long ORM chains and
comprehensions with nested lookups are the usual offenders.

```python
# Bad: black wraps this into a multi-line mess
result = (
    client.get_session()
    .query(User)
    .filter(User.active == True)
    .options(joinedload(User.orders))
    .order_by(User.created_at.desc())
    .limit(page_size)
    .all()
)

# Good: extract a helper so the call site stays clean
def get_active_users(session, page_size: int) -> list[User]:
    return (
        session.query(User)
        .filter(User.active == True)
        .options(joinedload(User.orders))
        .order_by(User.created_at.desc())
        .limit(page_size)
        .all()
    )

users = get_active_users(client.get_session(), page_size=20)
```

```python
# Bad: dict comprehension with inline chain — black expands to 5+ lines
config = {
    k: settings.get(k, defaults.get(k, fallbacks.get(k, None)))
    for k in required_keys
}

# Good: extract the lookup
def resolve_setting(key, settings, defaults, fallbacks):
    return settings.get(key, defaults.get(key, fallbacks.get(key)))

config = {k: resolve_setting(k, settings, defaults, fallbacks) for k in required_keys}
```
