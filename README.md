# blog

[![Build Status](https://github.com/ryuuinc/blog/workflows/Hexo/badge.svg)](https://github.com/ryuuinc/blog/actions)

## 配置主题

```Bash
git subtree add \
    --prefix themes/fluid \
    git@github.com:fluid-dev/hexo-theme-fluid.git master \
    --squash
```

## 使用方法

```Bash
npm i
npx hexo s
```

## 部署方法

由于使用了 `Github Actions`，且相关凭证保存在 `Secrets` 中，只需正常提交代码即可自动部署到服务器
