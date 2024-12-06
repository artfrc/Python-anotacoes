# Repositories
Ações feitas no BD.from sqlalchemy import create_engine

## Exemplo de repositório:

```python

```

## Exemplo teste de integração:
```python
import pytest
from src.models.sqlite.settings.connection import db_connection_handler
from .users_repository import UserRepository

db_connection_handler.connect_to_db()

@pytest.mark.skip(reason="Interacts with the database")
def test_list_users():
    repo = UserRepository(db_connection_handler)
    response = repo.list_users()

    response_expected = [
        {"id": 1, "name": "John Doe", "username": "johndoe"},
        {"id": 2, "name": "Jane Doe", "username": "janedoe"},
    ]

    response_as_dict = [
        {"id": user.id, "name": user.name, "username": user.username}
        for user in response
    ]

    assert response_as_dict == response_expected
```