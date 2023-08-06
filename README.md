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

## Закрытие подключения к базе данных:

```javascript
process.on("SIGINT", () => {
  pool.end();
  process.exit(0);
});
```

## Получение данных с использованием агрегатных функций (пример подсчета общего числа пользователей):

```javascript
app.get("/users/count", async (req, res) => {
  const query = "SELECT COUNT(*) AS user_count FROM users";

  try {
    const result = await pool.query(query);
    res.json(result.rows[0]);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Сортировка данных (пример получения пользователей, отсортированных по возрасту по убыванию):

```javascript
app.get("/users/sorted-by-age", async (req, res) => {
  const query = "SELECT * FROM users ORDER BY age DESC";

  try {
    const result = await pool.query(query);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Ограничение количества возвращаемых результатов (пример получения ограниченного числа пользователей):

```javascript
app.get("/users/limited", async (req, res) => {
  const limit = parseInt(req.query.limit, 10) || 10;
  const query = "SELECT * FROM users LIMIT $1";
  const values = [limit];

  try {
    const result = await pool.query(query, values);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование оператора IN для фильтрации по нескольким значениям (пример получения пользователей с определенными идентификаторами):

```javascript
app.get("/users/:ids", async (req, res) => {
  const ids = req.params.ids.split(",").map((id) => parseInt(id, 10));
  const query = "SELECT * FROM users WHERE id IN ($1)";
  const values = [ids];

  try {
    const result = await pool.query(query, values);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование функции LIKE для поиска данных с учетом подстроки (пример получения пользователей по части email):

```javascript
app.get("/users/email/:part", async (req, res) => {
  const part = `%${req.params.part}%`;
  const query = "SELECT * FROM users WHERE email LIKE $1";
  const values = [part];

  try {
    const result = await pool.query(query, values);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование оператора BETWEEN для фильтрации данных в заданном диапазоне (пример получения пользователей в определенном возрастовом диапазоне):

```javascript
app.get("/users/age-range/:min/:max", async (req, res) => {
  const minAge = parseInt(req.params.min, 10);
  const maxAge = parseInt(req.params.max, 10);
  const query = "SELECT * FROM users WHERE age BETWEEN $1 AND $2";
  const values = [minAge, maxAge];

  try {
    const result = await pool.query(query, values);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование агрегатных функций и GROUP BY для группировки данных (пример получения количества пользователей по возрасту):

```javascript
app.get("/users/count-by-age", async (req, res) => {
  const query = "SELECT age, COUNT(*) AS user_count FROM users GROUP BY age";

  try {
    const result = await pool.query(query);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование условий CASE WHEN для возврата различных значений в зависимости от условий (пример получения пользователей с указанием возрастной категории):

```javascript
app.get("/users/age-category", async (req, res) => {
  const query = `
    SELECT
      name,
      age,
      CASE
        WHEN age <= 18 THEN 'Underage'
        WHEN age BETWEEN 19 AND 35 THEN 'Young Adult'
        WHEN age BETWEEN 36 AND 50 THEN 'Middle-aged Adult'
        ELSE 'Senior'
      END AS age_category
    FROM
      users
  `;

  try {
    const result = await pool.query(query);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование подзапросов (пример получения пользователей с профилем, удовлетворяющим определенным условиям):

```javascript
app.get("/users/with-specific-profile", async (req, res) => {
  const query = `
    SELECT * FROM users
    WHERE EXISTS (
      SELECT 1 FROM profiles
      WHERE profiles.user_id = users.id
      AND profiles.bio LIKE '%Node.js%'
    )
  `;

  try {
    const result = await pool.query(query);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование оператора JOIN для связи данных из нескольких таблиц по условию (пример получения пользователей и их заказов):

```javascript
app.get("/users-with-orders", async (req, res) => {
  const query = `
    SELECT users.id, users.name, orders.order_id, orders.total_amount
    FROM users
    LEFT JOIN orders ON users.id = orders.user_id
  `;

  try {
    const result = await pool.query(query);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование функции DISTINCT для получения уникальных значений (пример получения уникальных возрастов пользователей):

```javascript
app.get("/users/unique-ages", async (req, res) => {
  const query = "SELECT DISTINCT age FROM users";

  try {
    const result = await pool.query(query);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование агрегатной функции SUM для подсчета суммы числовых значений (пример получения общей суммы всех заказов):

```javascript
app.get("/orders/total-amount", async (req, res) => {
  const query = "SELECT SUM(total_amount) AS total_amount FROM orders";

  try {
    const result = await pool.query(query);
    res.json(result.rows[0]);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование функции COALESCE для возврата первого ненулевого значения из списка (пример получения пользователей с указанием их реального имени, если оно задано, или имени пользователя в противном случае):

```javascript
app.get("/users/real-name-or-username", async (req, res) => {
  const query = `
    SELECT id, COALESCE(real_name, name) AS display_name
    FROM users
  `;

  try {
    const result = await pool.query(query);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование функции DATE_TRUNC для группировки данных по временному интервалу (пример получения количества заказов по месяцам):

```javascript
app.get("/orders/count-by-month", async (req, res) => {
  const query = `
    SELECT DATE_TRUNC('month', order_date) AS month, COUNT(*) AS order_count
    FROM orders
    GROUP BY month
    ORDER BY month
  `;

  try {
    const result = await pool.query(query);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование оконных функций для аналитических вычислений (пример получения ранжированного списка пользователей по возрасту):

```javascript
app.get("/users/rank-by-age", async (req, res) => {
  const query = `
    SELECT id, name, age,
           RANK() OVER (ORDER BY age) AS age_rank
    FROM users
  `;

  try {
    const result = await pool.query(query);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование подзапросов с оператором IN и EXISTS для фильтрации данных (пример получения пользователей, у которых есть заказы):

```javascript
app.get("/users/with-orders", async (req, res) => {
  const query = `
    SELECT id, name, email
    FROM users
    WHERE id IN (SELECT DISTINCT user_id FROM orders)
  `;

  try {
    const result = await pool.query(query);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование временных функций (пример получения заказов за последние 7 дней):

```javascript
app.get("/orders/last-7-days", async (req, res) => {
  const query = `
    SELECT *
    FROM orders
    WHERE order_date >= NOW() - INTERVAL '7 days'
  `;

  try {
    const result = await pool.query(query);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование полнотекстового поиска с помощью оператора @@ (пример поиска пользователей по текстовым полям):

```javascript
app.get("/users/search/:query", async (req, res) => {
  const searchQuery = req.params.query;
  const query = `
    SELECT *
    FROM users
    WHERE name @@ to_tsquery($1)
       OR email @@ to_tsquery($1)
       OR profile @@ to_tsquery($1)
  `;

  try {
    const result = await pool.query(query, [searchQuery]);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование оператора RETURNING для получения обновленных данных после выполнения запроса (пример обновления данных пользователя и возврата обновленной записи):

```javascript
app.put("/users/:id", async (req, res) => {
  const userId = req.params.id;
  const { name, age, email } = req.body;
  const query =
    "UPDATE users SET name = $1, age = $2, email = $3 WHERE id = $4 RETURNING *";
  const values = [name, age, email, userId];

  try {
    const result = await pool.query(query, values);
    if (result.rows.length === 0) {
      return res.status(404).json({ error: "User not found" });
    }
    res.json(result.rows[0]);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование INNER JOIN для объединения данных из двух таблиц по ключу (пример получения пользователей и информации о их заказах):

```javascript
app.get("/users-with-orders", async (req, res) => {
  const query = `
    SELECT users.id, users.name, orders.order_id, orders.total_amount
    FROM users
    INNER JOIN orders ON users.id = orders.user_id
  `;

  try {
    const result = await pool.query(query);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```

## Использование LEFT JOIN для объединения данных из двух таблиц и возврата всех строк из левой таблицы, даже если нет совпадающих значений в правой таблице (пример получения пользователей и информации о заказах, если она доступна):

```javascript
app.get("/users-with-orders", async (req, res) => {
  const query = `
    SELECT users.id, users.name, orders.order_id, orders.total_amount
    FROM users
    LEFT JOIN orders ON users.id = orders.user_id
  `;

  try {
    const result = await pool.query(query);
    res.json(result.rows);
  } catch (error) {
    res.status(500).json({ error: "Internal Server Error" });
  }
});
```
