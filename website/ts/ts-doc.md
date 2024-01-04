

## 安装

```bash
npm install -D typedoc

#版本查看
npx typedoc --version

#手动执行
npx typedoc --entryPoints ./src/index.ts --out ./dist

```

## 配置



```json
{
    "$schema": "https://typedoc.org/schema.json",
    "entryPoints": ["./src/index.ts","./src/app.ts"],
    "out": "./dist/doc"
}
// npx typedoc --options ./typedoc.json

// 可以直接使用 tsconfig.json
// npx typedoc --tsconfig ./tsconfig.json  和上面一样的效果
```

插件转化为markdown

```sh
npm i -D typedoc-plugin-markdown
npx typedoc --options ./typedoc.json
```

