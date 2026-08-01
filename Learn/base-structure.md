# Neovim 项目主要文件结构

这个 Neovim 项目的主要文件结构大致是：

```text
/home/allworldg/neovim
├── src/                 # Neovim 核心源码，主要是 C / Lua / Zig
│   ├── nvim/            # 主体编辑器实现
│   │   ├── api/         # public API 实现
│   │   ├── eval/        # Vimscript 表达式求值相关
│   │   ├── event/       # 事件循环、异步事件
│   │   ├── lua/         # Lua 集成
│   │   ├── msgpack_rpc/ # RPC 通信
│   │   ├── os/          # OS 抽象层
│   │   ├── tui/         # 终端 UI
│   │   ├── viml/        # Vimscript 解析相关
│   │   └── vterm/       # terminal emulator 相关
│   ├── cjson/           # bundled cjson
│   ├── klib/            # 小型 C 工具库
│   ├── mpack/           # msgpack 支持
│   ├── xdiff/           # diff 算法相关
│   ├── xxd/             # xxd 工具
│   └── tee/             # tee 辅助程序
│
├── runtime/             # 运行时文件，用户使用 nvim 时加载
│   ├── autoload/        # Vimscript autoload 脚本
│   ├── colors/          # 配色方案
│   ├── compiler/        # 编译器配置
│   ├── doc/             # 帮助文档
│   ├── ftplugin/        # filetype 插件
│   ├── indent/          # 缩进规则
│   ├── lua/             # runtime Lua 模块，如 vim.*
│   ├── plugin/          # 默认插件
│   ├── queries/         # treesitter queries
│   ├── syntax/          # 语法高亮规则
│   └── tutor/           # 教程文件
│
├── test/                # 测试
│   ├── functional/      # 功能测试，覆盖 API、UI、Lua、terminal 等
│   ├── unit/            # 单元测试
│   ├── benchmark/       # 性能基准测试
│   ├── client/          # 测试客户端/RPC 辅助代码
│   └── old/             # 旧 Vim 测试套件
│
├── cmake/               # CMake helper 脚本和 Find*.cmake
├── cmake.config/        # 生成 config/version/path 头文件的模板
├── cmake.deps/          # 依赖构建相关 CMake
├── cmake.packaging/     # 打包相关配置、图标、安装脚本
├── deps/                # 第三方依赖源码/配置
├── scripts/             # 维护、发布、lint、同步 Vim patch 的脚本
├── contrib/             # 贡献者辅助工具，如 shell completion、gdb 脚本
├── build/               # 本地构建输出目录
├── .deps/               # 本地依赖构建输出目录
├── .github/             # GitHub Actions、issue 模板等
│
├── CMakeLists.txt       # 主 CMake 构建入口
├── Makefile             # 常用构建命令封装
├── build.zig            # Zig 构建入口
├── build.zig.zon        # Zig package 配置
├── README.md            # 项目说明
├── BUILD.md             # 构建说明
├── INSTALL.md           # 安装说明
├── CONTRIBUTING.md      # 贡献指南
├── MAINTAIN.md          # 维护者文档
└── AGENTS.md            # 本仓库给 AI agent 的说明
```

核心可以这样理解：

- `src/nvim/` 是编辑器本体，大部分核心逻辑在这里。
- `runtime/` 是随 Neovim 分发、运行时加载的脚本和资源。
- `test/` 是测试体系，`functional/` 最重要，覆盖用户可见行为。
- `cmake*`、`Makefile`、`build.zig*` 是构建系统。
- `build/` 和 `.deps/` 是本地生成目录，不是主要源码。
