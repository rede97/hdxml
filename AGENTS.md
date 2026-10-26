# hdxml — SystemVerilog RTL 分析器

[autowire](https://github.com/rede97/autowire) 的 RTL 分析 sidecar（由 stune 迁移裁剪而来）。2.6.0 起从 autowire 仓库拆出为独立项目，独立版本、独立 release；autowire 发布时按自己钉住的 hdxml 版本下载本仓库 release 二进制，不再编译。唯一功能：`analysis`——扫描 `.v`/`.sv`，提取模块层级/参数/端口/例化，导出 RtlIndex XML 目录（每源文件一个 XML + index.xml），供 autowire 只读消费。

## 文档（在 autowire 仓库 `docs/hdxml/`）

RtlIndex 是 hdxml 与 autowire 的接口契约，文档留在消费方仓库并随 `autowire.js` 打包。代码注释里的 `docs/hdxml/...` 都指 autowire 仓库的这些文件。

| 文档 | 内容 |
|---|---|
| [`docs/hdxml/rtlindex-xml.md`](https://github.com/rede97/autowire/blob/main/docs/hdxml/rtlindex-xml.md) | **RtlIndex XML 格式约束（与 autowire 的格式约定；分析器已按此产出）** |
| [`docs/hdxml/module-info.md`](https://github.com/rede97/autowire/blob/main/docs/hdxml/module-info.md) | DesignDb / 模块信息模型 |
| [`docs/hdxml/cli.md`](https://github.com/rede97/autowire/blob/main/docs/hdxml/cli.md) | CLI 选项与行为 |
| [`docs/hdxml/testing.md`](https://github.com/rede97/autowire/blob/main/docs/hdxml/testing.md) | fetch / scan / smoke 与错误基线 |
| [`docs/workspace/toml.md`](https://github.com/rede97/autowire/blob/main/docs/workspace/toml.md) | autowire 工作区 toml（`analysis run` 把 .f / 宏映射成 hdxml CLI） |

改 CLI 或 RtlIndex 格式时：先在本仓库发版，再到 autowire 同步文档并抬高其钉住的 hdxml 版本（autowire `hdxml-version.txt`）。

## 开发命令

```bash
cargo build          # dev 构建
cargo test           # 单元测试
cargo run -- -w <rtl_dir> --tree   # 目录扫描 + 依赖树（无子命令）
./dist.sh            # 发布构建：bun 模式 cargo-zigbuild 钉 glibc 2.17（CentOS 7+），产出 hdxml-linux-x64.tar.xz
```

## 约定

- Rust edition 2024；`anyhow::Result` 于边界；doc 注释用中文。
- **Pin `sv-parser = 0.13.4`**——API 版本敏感。
- 并发：rayon 池 + DashMap/DashSet/Mutex；DesignDb 屏障后不可变（Arc 共享，只读无锁）。禁止对语法树 `unsafe impl Send/Sync`。
- 终端输出只走 ProgressCenter；功能代码不得直接 `println!`/indicatif。
- **span 基准 = 预处理后文本**；两阶段流程必须 `parse_sv_pp` + `PreprocessedText.origins` 做源映射（`docs/hdxml/module-info.md` §3 勘定）。
- 分析纯净性：DesignDb 无条件全量收集，零功能标志位。
- release profile：`opt-level="z"`、`lto`、`codegen-units=1`、`strip`。发布构建：Linux x64（`dist.sh`，glibc 2.17）、macOS arm64/x64、Windows x64（`windows-2022` 上 `--target x86_64-pc-windows-msvc` 的 `hdxml.exe`）。

## 发布

1. 改 `Cargo.toml` 的 `version`（同步 `Cargo.lock`：`cargo build`）。
2. 推 `main`，再人工 `git tag v<version> && git push origin v<version>`。
3. `.github/workflows/release.yml` 校验 tag = `v<Cargo.toml version>`，跑 `cargo test`，构建并发布 `hdxml-linux-x64.tar.xz`（glibc 2.17）、`hdxml-macos-arm64.tar.xz`、`hdxml-macos-x64.tar.xz`、`hdxml-windows-x64.zip`。已存在的 release 拒绝覆盖；CI 不建 tag。

## 测试语料

`tests/fetch.ts`（Bun）拉取语料与第三方项目，`tests/scan.ts` 全量扫描测试（逐项目产出索引 + 错误基线汇总），`tests/smoke.ts` 工具链与 oracle 冒烟。用法与错误基线分类见 autowire 仓库 [`docs/hdxml/testing.md`](https://github.com/rede97/autowire/blob/main/docs/hdxml/testing.md)。语料不入库（见 `.gitignore`）。
