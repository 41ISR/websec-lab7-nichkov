# Задание: Поиск и устранение уязвимостей в веб-приложении

## Выполнение задания

1. SQL инъекция в поле логина пользователя, если ввести ' UNION SELECT username, password FROM users --, то пользователь получит доступ ко всем логинам и паролям других пользователей.
Для фикса необходимо изменить следующий код в index.js:
```bash
query = `SELECT * FROM users WHERE username = '${username}' AND password = '${password}'`
```
На этот:
```bash
const query = `SELECT * FROM users WHERE username = ? AND password = ?`;
    db.get(query, [username, password], (err, user) => {
      if (err) {
        console.error(err.message);
        return res.status(500).json({ error: "Ошибка сервера" });
      }
      if (!user) {
        return res.status(401).json({ error: "Неверные данные" });
      }
      res.json({ message: "Успешный вход", user });
    });
```
2. SQL инъекция в выборке пользователей, если в бд будет пользователь с логином ' UNION SELECT username, password FROM users --, то пользователь получит доступ ко всем логинам и паролям других пользователей при выборке данных из таблицы.
Для фикса необходимо изменить следующий код в index.js:
```bash
app.get("/users", (req, res) => {
    const { search } = req.query
    const query = `SELECT * FROM users WHERE username LIKE '%${search}%'`
    db.all(query, (err, users) => {
        if (err) return res.status(500).json({ error: "Ошибка сервера" })
        res.json(users)
    })
})
```
На этот:
```bash
app.get("/users", (req, res) => {
    const { search } = req.query
    const query = `SELECT * FROM users WHERE username LIKE ?`;
    const searchPattern = `%${search}%`;
    db.all(query, [searchPattern], (err, users) => { 
        if (err) return res.status(500).json({ error: "Ошибка сервера" })
        res.json(users) 
    });
})
```
3. SQL инъекция в вставке пользователей в таблицу, если в поле логина вписать ' UNION SELECT username, password FROM users --, то добавится пользователь с таким запросом, что может вызвать ошибки или иные проблемы.
Для фикса необходимо изменить следующий код в index.js:
```bash
app.post("/messages", (req, res) => {
    const { user_id, content } = req.body
    db.run(
        `INSERT INTO messages (user_id, content) VALUES (${user_id}, '${content}')`,
        function (err) {
            if (err) return res.status(500).json({ error: "Ошибка сервера" })
            res.json({ message: "Сообщение отправлено", id: this.lastID })
        }
    )
})
```
На этот:
```bash
db.run(
    `INSERT INTO messages (user_id, content) VALUES (?, ?)`,
    [user_id, content],
    function (err) {
        if (err) return res.status(500).json({ error: "Ошибка сервера" });
        res.json({ message: "Сообщение отправлено", id: this.lastID });
    }
);

```

4. SQL инъекция в обновлении имени пользователя, если в поле вписать ' UNION SELECT username, password FROM users --, то в бд добавится пользователь с таким запросом, что может вызвать ошибки или иные проблемы.
Для фикса необходимо изменить следующий код в index.js:
```bash
app.post("/users/update", (req, res) => {
    const { id, username } = req.body
    db.run(
        `UPDATE users SET username = '${username}' WHERE id = ${id}`,
        function (err) {
            if (err) return res.status(500).json({ error: "Ошибка сервера" })
            res.json({ message: "Имя пользователя обновлено" })
        }
    )
})
```
На этот:
```bash
db.run(
    `UPDATE users SET username = ? WHERE id = ?`,
    [username, id],
    function (err) {
        if (err) return res.status(500).json({ error: "Ошибка сервера" });
        res.json({ message: "Имя пользователя обновлено" });
    }
);
```

5. XSS инъекция в messages, если в поле сообщения вписать любую js команду, то она выполнится на странице.
Для фикса необходимо изменить следующий код в messages.tsx:
```bash
dangerouslySetInnerHTML={{ __html: msg.content }}
```
На этот:
```bash
<div
    key={index}
    className="p-1 border-b last:border-none"
>
    {msg.content}
</div>
```

6. Добавил хэширование паролей с помощью bycrypt.
