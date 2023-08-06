## Подключение к базе данных:

```javascript
const { Pool } = require("pg");

const pool = new Pool({
  user: "postgres",
  password: "password",
  host: "localhost",
  port: 5432,
  database: "mydatabase",
});
```

## Пример обработки GET-запроса для получения данных из таблицы:

```javascript
app.get("/users", (req, res) => {
  pool.query("SELECT * FROM users", (err, result) => {
    if (err) {
      console.error(err);
      res.status(500).json({ error: "Internal server error" });
    } else {
      res.json(result.rows);
    }
  });
});
```

## Пример обработки POST-запроса для добавления данных в таблицу:

```javascript
app.post("/users", (req, res) => {
  const { name, email } = req.body;

  pool.query(
    "INSERT INTO users (name, email) VALUES ($1, $2)",
    [name, email],
    (err, result) => {
      if (err) {
        console.error(err);
        res.status(500).json({ error: "Internal server error" });
      } else {
        res.sendStatus(201);
      }
    }
  );
});
```

## Пример обработки PUT-запроса для обновления данных в таблице:

```javascript
app.put("/users/:id", (req, res) => {
  const id = req.params.id;
  const { name, email } = req.body;

  pool.query(
    "UPDATE users SET name = $1, email = $2 WHERE id = $3",
    [name, email, id],
    (err, result) => {
      if (err) {
        console.error(err);
        res.status(500).json({ error: "Internal server error" });
      } else {
        res.sendStatus(200);
      }
    }
  );
});
```

## Пример обработки DELETE-запроса для удаления данных из таблицы:

```javascript
app.delete("/users/:id", (req, res) => {
  const id = req.params.id;

  pool.query("DELETE FROM users WHERE id = $1", [id], (err, result) => {
    if (err) {
      console.error(err);
      res.status(500).json({ error: "Internal server error" });
    } else {
      res.sendStatus(200);
    }
  });
});
```

## Выполнение запроса с параметрами:

```javascript
app.get("/users/:id", (req, res) => {
  const id = req.params.id;

  pool.query("SELECT * FROM users WHERE id = $1", [id], (err, result) => {
    if (err) {
      console.error(err);
      res.status(500).json({ error: "Internal server error" });
    } else {
      res.json(result.rows[0]);
    }
  });
});
```

## Обработка ошибок при выполнении запроса:

```javascript
app.get("/users", (req, res) => {
  pool.query("SELECT * FROM users", (err, result) => {
    if (err) {
      console.error(err);
      res.status(500).json({ error: "Internal server error" });
    } else {
      res.json(result.rows);
    }
  });
});
```

## Использование транзакций:

```javascript
app.post("/users", (req, res) => {
  const { name, email } = req.body;

  pool.connect((err, client, done) => {
    if (err) {
      console.error(err);
      res.status(500).json({ error: "Internal server error" });
    }

    try {
      client.query("BEGIN");

      const queryText = "INSERT INTO users (name, email) VALUES ($1, $2)";
      const values = [name, email];

      client.query(queryText, values, (err, result) => {
        if (err) {
          console.error(err);
          client.query("ROLLBACK");
          res.status(500).json({ error: "Internal server error" });
        } else {
          client.query("COMMIT");
          res.sendStatus(201);
        }
      });
    } finally {
      done();
    }
  });
});
```

## Использование пакета `pg-promise` для выполнения запросов к базе данных:

```javascript
const pgp = require("pg-promise")();
const db = pgp("postgres://username:password@localhost:5432/mydatabase");

app.get("/users", (req, res) => {
  db.any("SELECT * FROM users")
    .then((data) => res.json(data))
    .catch((err) => {
      console.error(err);
      res.status(500).json({ error: "Internal server error" });
    });
});

app.post("/users", (req, res) => {
  const { name, email } = req.body;

  db.none("INSERT INTO users (name, email) VALUES ($1, $2)", [name, email])
    .then(() => res.sendStatus(201))
    .catch((err) => {
      console.error(err);
      res.status(500).json({ error: "Internal server error" });
    });
});
```

## Объединение таблиц:

```javascript
app.get("/users", (req, res) => {
  pool.query(
    "SELECT users.*, addresses.city FROM users JOIN addresses ON users.address_id = addresses.id",
    (err, result) => {
      if (err) {
        console.error(err);
        res.status(500).json({ error: "Internal server error" });
      } else {
        res.json(result.rows);
      }
    }
  );
});
```

## Вставка нескольких записей с использованием `INSERT INTO ... SELECT`:

```javascript
app.post("/users/migrate", (req, res) => {
  pool.query(
    "INSERT INTO new_users (name, email) SELECT name, email FROM old_users WHERE created_at > $1",
    [new Date("2022-01-01")],
    (err, result) => {
      if (err) {
        console.error(err);
        res.status(500).json({ error: "Internal server error" });
      } else {
        res.sendStatus(201);
      }
    }
  );
});
```

## Вызов хранимой процедуры:

```javascript
app.post("/users", (req, res) => {
  const { name, email } = req.body;

  pool.query("CALL create_user($1, $2)", [name, email], (err, result) => {
    if (err) {
      console.error(err);
      res.status(500).json({ error: "Internal server error" });
    } else {
      res.sendStatus(201);
    }
  });
});
```

## Обновление записей с использованием условия `WHERE IN`:

```javascript
app.put("/users", (req, res) => {
  const { ids, status } = req.body;

  pool.query(
    "UPDATE users SET status = $1 WHERE id IN ($2:csv)",
    [status, ids],
    (err, result) => {
      if (err) {
        console.error(err);
        res.status(500).json({ error: "Internal server error" });
      } else {
        res.sendStatus(200);
      }
    }
  );
});
```

## Пример использования пакета `pg-format` для создания безопасных запросов:

```javascript
const format = require("pg-format");

app.put("/users", async (req, res) => {
  const { ids, status } = req.body;

  try {
    const formattedIds = format("%L", ids);

    const query = {
      name: "update-users",
      text: `UPDATE users SET status = $1 WHERE id IN (${formattedIds})`,
      values: [status],
    };

    await pool.query(query);
    res.sendStatus(200);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: "Internal server error" });
  }
});
```
