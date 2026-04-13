# ObjectToCsv

```js
function objectsToCsv<T extends Record<string, string | number | null>>(objects: T[]): string {
  if (objects.length === 0) return '';

  const header = Object.keys(objects[0]).join(',');
  const rows = objects.map(obj =>
    Object.values(obj).map(value => (value === null ? '' : String(value))).join(',')
  );

  return [header, ...rows].join('\n');
}

// 使用例
const data = [
  { name: "Alice", age: 25, city: "New York" },
  { name: "Bob", age: 30, city: "Los Angeles" },
  { name: "Charlie", age: null, city: "Chicago" }
];

console.log(objectsToCsv(data));
```
