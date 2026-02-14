# 文派插件发布规范 v1.0

> 适用于所有通过文派云桥（updates.wenpai.net）分发的 WordPress 插件和主题。

## 1. 版本号规范

遵循 [Semantic Versioning 2.0.0](https://semver.org/)。

### Git Tag
- 格式：`v{MAJOR}.{MINOR}.{PATCH}`（带 `v` 前缀）
- 示例：`v1.0.0`、`v2.3.1`
- 预发布：`v1.0.0-beta.1`、`v1.0.0-rc.1`
- 禁止：大写 `V`、省略 patch（`v1.0`）、非数字后缀

### 插件主文件 `Version` Header
- 格式：`{MAJOR}.{MINOR}.{PATCH}`（不带 `v` 前缀）
- 必须与 Git Tag 数值部分完全一致

### readme.txt `Stable tag`
- 格式：`{MAJOR}.{MINOR}.{PATCH}`（不带 `v` 前缀）
- 必须与插件主文件 Version 完全一致

### 一致性校验
CI 强制校验三者一致，不一致则拒绝发布：
```
Git Tag v1.2.3 → Version: 1.2.3 → Stable tag: 1.2.3
```

## 2. 插件主文件 Header 必填字段

```php
<?php
/**
 * Plugin Name: 插件显示名称
 * Plugin URI:  https://wenpai.org/plugins/{slug}/
 * Description: 插件简短描述
 * Version:     1.0.0
 * Author:      作者名
 * Author URI:  https://作者网站/
 * License:     GPL v2 or later
 * Text Domain: {slug}
 * Domain Path: /languages
 * Update URI:  https://updates.wenpai.net
 * Requires at least: 6.0
 * Requires PHP: 7.4
 */
```

`Update URI` 是云桥的关键字段，WordPress 5.8+ 会据此触发 `update_plugins_updates.wenpai.net` filter。

## 3. ZIP 打包规范

### 文件名
- 格式：`{slug}-{version}.zip`
- 示例：`wpslug-1.0.0.zip`

### 目录结构
ZIP 内顶层目录必须等于插件 slug：
```
wpslug-1.0.0.zip
└── wpslug/
    ├── wpslug.php
    ├── readme.txt
    ├── includes/
    ├── assets/
    └── languages/
```

### 排除文件
ZIP 中不得包含：
```
.git/  .github/  .forgejo/  .gitignore  .gitattributes
node_modules/  vendor/（除非是生产依赖）
tests/  phpunit.xml  phpcs.xml  phpstan.neon
composer.json  composer.lock（除非运行时需要 autoload）
*.md（除 readme.txt 外）  .editorconfig  .env*
```

### SHA-256 校验
每个 ZIP 必须附带 SHA-256 校验值，写入 Release body 或单独的 `.sha256` 文件。

## 4. Forgejo Release 规范

### Release 创建
- 由 CI 自动创建，禁止手动创建
- Release name = Tag name（如 `v1.0.0`）
- Release body 包含 changelog + SHA-256

### Release Assets
每个 Release 必须包含：
1. `{slug}-{version}.zip` — 插件 ZIP 包
2. `{slug}-{version}.zip.sha256` — SHA-256 校验文件

### Release Body 模板
```markdown
## Changelog

- 变更内容 1
- 变更内容 2

## Checksums

| File | SHA-256 |
|------|---------|
| wpslug-1.0.0.zip | `7f9a2b6d...` |
```

## 5. plugin-registry 注册文件规范

每个插件在 `WenPai-org/plugin-registry` 仓库中有一个注册文件：

```json
{
  "$schema": "../schema.json",
  "slug": "wpslug",
  "name": "WPSlug",
  "type": "plugin",
  "repo": "WenPai-org/wpslug",
  "main_file": "wpslug.php",
  "description": "Advanced slug management plugin",
  "homepage": "https://wpslug.com",
  "author": "WPSlug.com",
  "license": "GPL-2.0-or-later",
  "requires_wp": "6.0",
  "requires_php": "7.4",
  "tested_wp": "6.7.1",
  "tags": ["slug", "seo", "pinyin"],
  "icons": {
    "1x": "https://updates.wenpai.net/assets/plugins/wpslug/icon-128x128.png",
    "2x": "https://updates.wenpai.net/assets/plugins/wpslug/icon-256x256.png"
  }
}
```

## 6. CI Workflow 标准

所有插件使用统一的 CI 模板（来自 `WenPai-org/ci-templates` 仓库）。

### 触发条件
- Push tag `v*` → 自动构建 + 发布 Release

### 流程
1. Checkout 代码
2. 校验版本一致性（tag vs header vs readme）
3. 运行 PHP lint
4. 构建 ZIP（排除开发文件，顶层目录 = slug）
5. 计算 SHA-256
6. 创建 Forgejo Release + 上传 assets
