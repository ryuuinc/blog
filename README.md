### 使用方法

```Bash
yarn install
cd themes/suka
yarn install --production
cd -
npx hexo s
```

### 部署方法

由于使用了 `Github Actions`，且相关凭证保存在 `Secrets` 中，只需正常提交代码即可自动部署到服务器
