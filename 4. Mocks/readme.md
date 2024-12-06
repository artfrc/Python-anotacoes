# Mocks
-*Objetos simulados que você cria em seus testes para substituir dependências reais de forma controlada.*
-*Garantem que o teste avalie apenas a lógica da unidade em análise, sem depender de componentes externos.*

---

## Exemplo de mock

### Repository
```python
from sqlite3 import Connection

class UserRepository:
    def __init__(self, conn: Connection):
        self.__conn = conn

    def registry_user(self, username: str, password: str):
        cursor = self.__conn.cursor()
        cursor.execute(
            '''
                INSERT INTO users 
                (username, password, balance) 
                VALUES 
                    (?, ?, ?);
            ''',
            (username, password, 0)
        )
        self.__conn.commit()
```

## Como criar teste unitário com Mock disso?
*Mock aqui é interessante para não mexer com o banco real*

1. Analisar quais elementos interagem com o banco. Temos dois:
- cursor: usando __execute__ e o __fetchone__.
- conn: usando para gerar o cursor e para fazer o __commit__.

    Logo, criaremos um Mock de cada com essses elementos.

```python
class MockCursor:
    def __init__(self):
        self.execute = Mock()
        self.fetchone = Mock()

class MockConn:
    def __init__(self):
        self.cursor = Mock(return_value = MockCursor())
        self.commit = Mock()
```

### *self.cursor = Mock(return_value = MockCursor())*
Ao criar um cursor com o MockConn iremos retornar uma instância do MockCursor.

- No teste, criar uma conexão mocada para não ir no bd real, criamos um repository com a conexão mocada e testar o **_register_user_**

```python
from src.models.sqlite.repositories.user_repository import UserRepository
from src.models.sqlite.settings.db_connection_handler import db_connection_handler
from unittest.mock import Mock

class MockCursor:
    def __init__(self):
        self.execute = Mock()
        self.fetchone = Mock()

class MockConn:
    def __init__(self):
        self.cursor = Mock(return_value = MockCursor())
        self.commit = Mock()

def test_repository(): # teste

    username  = 'JohnDoe'
    password = '12345'

    mock_conn = MockConn() # conexão mocada
    repo = UserRepository(mock_conn) # repositório
    repo.registry_user(username, password)

    # verificar se a função foi executada corretamente
    cursor = mock_conn.cursor.return_value # cria o cursor para executar os outros comandos

    # Checa linha por linha da consulta feita no execute do repository
    assert "INSERT INTO users" in cursor.execute.call_args[0][0]
    assert "(username, password, balance)" in cursor.execute.call_args[0][0] 
    assert "VALUES" in cursor.execute.call_args[0][0]
    assert cursor.execute.call_args[0][1] == (username, password, 0)
```

### **Repositório completo com testes e mocks [aqui](https://github.com/artfrc/Seguranca/tree/main/src/models/sqlite/repositories)**