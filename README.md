# A2LTool

A tool to edit, merge, update and **generate** A2L (ASAP2) files.

基于 [DanielT/a2ltool](https://github.com/DanielT/a2ltool) v3.2.2 二次开发，新增 YAML 配置支持。

---

**作者**: Wanghaitao | **邮箱**: wht_0117@163.com

**开始时间**: 2026-05-22 | **版本**: 1.0.0

---

## Features

- 从 YAML 配置文件生成 A2L 文件 (`--from-yaml`)
- 生成 YAML 配置示例模板 (`--yaml-example`)
- 支持 GROUP `root` / `sub_groups` 层级结构
- 完整的 TAB_VERB / TAB_NOINTP 查表类型支持
- 基于 ELF/PDB 文件更新观测量和标定量的地址
- 合并多个 A2L 文件为单个文件
- 从 ELF 文件添加新的观测量或标定量
- 一致性检查 (`--check`)
- 显示 A2L 文件中嵌入的 XCP 连接参数
- 维护 A2L 文件中项目的格式和顺序，使原始文件与更新/修改文件之间的差异尽可能小
- 支持 A2L 版本 1.71（最新）

## 安装

### 从源码编译

```bash
cargo build --release
```

编译后的二进制文件位于 `target/release/a2ltool.exe`。

## 用法

### 从 YAML 配置生成 A2L 文件

```bash
# 从 YAML 配置文件生成 A2L 文件并进行一致性检查和排序
a2ltool --from-yaml config.yaml --check --sort -o output.a2l

# 生成一份完整的 YAML 配置示例模板
a2ltool --yaml-example -o example.yaml
```

详细的 YAML 配置说明请参考 [A2L_YAML_CONFIG_SPEC.md](A2L_YAML_CONFIG_SPEC.md)。

### 合并 A2L 文件

```bash
a2ltool file1.a2l --merge file2.a2l -o merged.a2l
```

### 更新 A2L 文件中的地址

```bash
a2ltool input.a2l --elffile firmware.elf --update -o updated.a2l
```

### 创建新的 A2L 文件

```bash
a2ltool --create -o newfile.a2l
```

### 一致性检查

```bash
a2ltool input.a2l --check --strict
```

更多详细用法请运行 `a2ltool --help`。

## 关于 A2L 文件

A2L 文件描述了嵌入式设备（通常是汽车 ECU）的测量变量和可调参数。

A2L 文件的消费者通常允许通过 XCP 等协议进行在线标定和/或通过生成可刷写的参数集进行离线调校。市面上有多种商业工具可用于此目的。

A2L 文件格式由 ASAM 制定，正式名称为 ASAM MCD-2 MC。

## License

Licensed under either of [MIT](LICENSE-MIT) or [Apache-2.0](LICENSE-APACHE) at your option.
