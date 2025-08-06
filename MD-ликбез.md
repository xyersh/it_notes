
Первый параграф

Второй параграф. Продолжение                                  второго параграфа. <br/>Третий параграф


#### Заголовки:
- # `# Заголовок первого уровня`
- ## `## Заголовок второго уровня`
- ### `### Заголовок третьего уровня`

**Жирный и курсив:**
- **Жирный текст:** `**жирный текст**` или `__жирный текст__`
- *Курсивный текст:* `*курсивный текст*` или `_курсивный текст_`
- ***Курсивный жирный текст:*** This is really***very***important text
- ~~Зачеркнутый текст~~: `~~зачеркнутый текст~~`
- ==Подсвеченный текст==: `==подсвеченный текст==`

#### Списки:
- **Нумерованный список:**
    1. Первый пункт
    2. Второй пункт
- **Маркированный список:**
    - Первый пункт
    - Второй пункт
- **Вложенный список:** 
1. First list item 
	1. Ordered nested list item 
2. Second list item 
	- Unordered nested list item

#### Ссылки:
- **Внутренние ссылки:**  `[[Название заметки]]`   [[ACID]]
- **Внешние ссылки:**  `[Текст ссылки](https://ссылка)`[google.com](https://google.com)
- **Ссылки на блоки**  `[[Название заметки#^идентификатор блока|alias]]`  
[[MD-ликбез#^123|README]] - где `MD-ликбез` - название файла, `#^123` - якорь на конкретное место в файле, `|README` - название ссылки, которое будет видеть пользователь.

#### Изображения:**
- `![Описание изображения](путь_к_изображению)`
     Примеры:
 -  `![локальная картинка:](./refs/golang.png)`
	     ![локальная картинка:](golang.png)
 - `![интернет ссылка:](https://www.google.com/url?sa=i&url=https%3A%2F%2Fgithub.com%2Fgolang%2Fgo&psig=AOvVaw3JxzeH5l3SX6gCK44B9XO1&ust=1736116171874000&source=images&cd=vfe&opi=89978449&ved=0CBQQjRxqFwoTCLjItPGO3YoDFQAAAAAdAAAAABAE)`
	  ![интернет картинка:](https://camo.githubusercontent.com/ff89c51c9e5a3de2b752b37bf6ab32401b9649d7acb1633ece9a40c85ae28b95/68747470733a2f2f676f6c616e672e6f72672f646f632f676f706865722f6669766579656172732e6a7067)


#### Подсветка кода
```javascript
function greet(name) {
  console.log("Hello, " + name + "!");
}
```

```go
type SomeType struct {
	Name string,
	Surbane string
}
```


#### Цитаты
- **Цитаты:**  `> сождержимое цитаты`
> Dorothy followed her through many of the beautiful rooms in her castle.

- **многострочные цитаты на несколько параграфов:**
> Dorothy followed her through many of the beautiful rooms in her castle.
>
> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.

- **Вложенные цитаты:**
> Dorothy followed her through many of the beautiful rooms in her castle.
>
>> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.

####  Список задач:
	- [ ] сделать то
	- [ ] сделать это
	- [ ] ничего не делать

- **Сноски:**
This is a simple footnote[^1].

#### Комментарии: 
комментарии видны только  в режиме редактирования.

This is an %%inline%% comment. 
%% 
This is a block comment. 
Block comments can span multiple lines.
%%


#### экранирование символов:  ^123
- Asterisk: `\*`
- Underscore: `\_`
- Hashtag: `\#`
- Backtick: `` \` ``
- Pipe (used in tables): `\|`
- Tilde: `\~` 

[^1]: This is the referenced text. 
[^2]: Add 2 spaces at the start of each new line. This lets you write footnotes that span multiple lines.
[^note]: Named footnotes still appear as numbers, but can make it easier to identify and link references.


#### Закрывающиеся блоки
```xml
<details>
<summary>TLDR</summary>
Основы кластера Kafka — это продюсер, брокер и консумер. Продюсер пишет сообщения в лог брокера, а консумер его читает.
</details>
```

<details>
<summary>TLDR</summary>
Основы кластера Kafka — это продюсер, брокер и консумер. Продюсер пишет сообщения в лог брокера, а консумер его читает.
</details>
