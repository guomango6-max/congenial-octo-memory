# Mango Creator (DSH agent preset)

两个 bundle,做的是同一件事的两种方式。**按你的需求("改造 creator 让它适配")用
`mango-creator-override/`。**

| 目录 | 做法 | `config.id` | 结果 |
|---|---|---|---|
| `mango-creator-override/` | **override** 出厂的 `preset-cordis` 行 | 保持 `cordis` | Creator mode 本身变成 Mango Creator。已有会话不受影响。 |
| `mango-creator/` | **insert** 一条新声明 | `mango-creator` | 多出一个并存的 preset,Creator mode 原样保留。 |

两者都是标准的 DSH preset bundle:一个 `package.json` 声明
`dsh.bundle.patch`,加一个 `cordis.patch.yml`。

## 装

桌面版在应用里开 Creator mode 对话:

```
plugin_manager → action: install_bundle → target: <目录的绝对路径>
```

它自己做包安装和 bundle 选择,不要用 shell 手动走。

桌面版**不能走命令行**:Electron 独占 `$DSH_HOME/profiles/desktop`,
`apps/desktop/README.md:146` 明确写了「The CLI cannot boot this profile」,
npm 装的 `dsh` 也改不了它。

装完 `plugin_manager list_plugins` 看对应 `preset-*` 行的激活状态,或直接看
Plugins 页。激活失败的声明会留在列表里带诊断。**已有会话保留它启动时的版本,
要新开会话才生效。**

## 两条贯穿始终的约束

**`config.id` 是会话身份,不是显示名。** 会话日志记录 `config.id`;恢复时按它
找定义,找不到直接拒绝恢复
(`packages/preset/agent-preset-registry/README.md`)。所以 override 那份把
`config.id` 保持成 `cordis`。

**改名靠 `config.name`。** 出厂 preset 不带 `name` 字段,"Creator" 来自 locale
字典;`display.ts:53` 的 `isBuiltInPreset` 要求 `name === undefined`,所以一旦
声明了 `config.name`,就走自己的显示名。副作用:卡片从 built-in 组挪到 custom
组,内置的那个用法说明弹窗消失。

## 不在 preset 层的东西

权限/审批(`sandbox-policy`、`user-approval`、`permission-presets`)都在
`base` bundle 的 host 层,改 preset 碰不到。而且出厂默认
(`workspace-write` + `ask`)就是「读放开,写和 shell 要批」,无需配置。
临时切:`/permission read-only|workspace-write|danger-full-access`。

同理 `office-kit` / `skill-office`(web-app bundle)和 `web-search-deepseek` /
`web-fetch-http`(base bundle)都已挂在 host 层,preset 这边只需要 `tool-office`
和 `tool-web` 两行。

## 校验状态

两份 patch 的 YAML(含 `!!js` 标签)均解析通过,插件名已逐一核对
`deepseek-ai/deepseek-harness` master 的 383 个包。未在真实 DSH 里安装运行过。
