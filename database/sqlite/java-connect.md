# python 连接sqlite



python3.11 版本自带sqlite

参考官方文档位置：https://docs.python.org/3/library/sqlite3.html

```
import sqlite3
con = sqlite3.connect("tutorial.db")
cur = con.cursor()

cur.execute("CREATE TABLE movie(title, year, score)")

res = cur.execute("SELECT name FROM sqlite_master")
#res.fetchone()
title, year = res.fetchone()
print(f'The highest scoring Monty Python movie is {title!r}, released in {year}')

res = cur.execute("SELECT name FROM sqlite_master WHERE name='spam'")
res.fetchone() is None


cur.execute("""
    INSERT INTO movie VALUES
        ('Monty Python and the Holy Grail', 1975, 8.2),
        ('And Now for Something Completely Different', 1971, 7.5)
""")
con.commit()

res = cur.execute("SELECT score FROM movie")
res.fetchall()

data = [
    ("Monty Python Live at the Hollywood Bowl", 1982, 7.9),
    ("Monty Python's The Meaning of Life", 1983, 7.5),
    ("Monty Python's Life of Brian", 1979, 8.0),
]
cur.executemany("INSERT INTO movie VALUES(?, ?, ?)", data)
con.commit()  
```

