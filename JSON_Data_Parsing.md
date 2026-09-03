# **Description**

You're given a list of dictionaries, where each dictionary represents a user with `id`, `name`, and `age` fields. Your task is to implement three API endpoints (functions) to query this data.

```json
[
  {"id": 1, "name": "Alice", "age": 25},
  {"id": 2, "name": "Bob", "age": 30},
  {"id": 3, "name": "Charlie", "age": 25},
  {"id": 4, "name": "David", "age": 40},
  {"id": 5, "name": "Eve", "age": 30}
]
```

1. **`get_user_by_id(users_data, user_id)`**: Returns the user dictionary with the matching `id`. If no user is found, return `None`.
2. **`search_users_by_name(users_data, query)`**: Returns a list of all user dictionaries where the `name` contains the `query` string (case-insensitive).
3. **`group_users_by_age(users_data)`**: Returns a dictionary where keys are ages and values are lists of user dictionaries for that age.
