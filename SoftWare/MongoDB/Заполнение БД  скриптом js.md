#### 1. создать скрипт
```js
const names = ['Alex', 'Bob', 'Frank', 'Dmitry', 'Julia', 'Mark', 'Vigor'];
const cities = ['Minsk', 'Brest', 'Grodno', 'Brest', 'Moskow', 'Berlin', 'London'];

const bulkData = [];

for (let i=0; i < 10000000; i++){
    bulkData.push({
        name: names[Math.floor(Math.random() * names.length)] + '_' + i,
        age: Math.floor(Math.random() * 60) + 18,
        verified: Math.random() < 0.5,
        email: `user${i}@example.com`,
        address: {
            city: cities[Math.floor(Math.random() * cities.length)],
            street: `Street ${Math.floor(Math.random() * 100 ) + 1}`
        }
    });

    if (bulkData.length === 1000 ) {
        db.users2.insertMany(bulkData);
        bulkData.length = 0;
        console.log(`insterted  ${i+1} documents`);
    }
}
```


#### 2.  сохранить скрипт

#### 3. запустить скрипт из окружения текущей БД
```js
	load('[ABS_PATH_TO_SCRIPT]')
```
	