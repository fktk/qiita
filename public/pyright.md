---
title: Neovimのpyrightに仮想環境を認識させる3つの方法
tags:
  - Python
  - neovim
  - pyright
  - nvim-lsp
private: true
updated_at: '2024-12-22T21:18:49+09:00'
id: 4e65fcb7cb116065c6aa
organization_url_name: null
slide: false
ignorePublish: false
---

NeovimでPython開発を行う際、LSPとしてpyrightを使用している方も多いと思います。
しかし、プロジェクトごとに仮想環境を設定している場合、pyrightがその仮想環境を正しく認識しないことがあります。

本記事では、その問題を解決するための3つの方法を紹介します。

### 前提

私は普段[uv](https://github.com/astral-sh/uv)を使ってプロジェクトを管理しています。
uvではvenvを使用して仮想環境が作られるので、venvを使用しているなら、本記事で紹介する方法が使えると思います。

また、Neovimの設定は[kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim)を参考しており、luaで記述しています。
LSPに関連するプラグインとして、以下のものを使用しています。

- neovim/nvim-lspconfig
- williamboman/mason.nvim
- williamboman/mason-lspconfig.nvim
- WhoIsSethDaniel/mason-tool-installer.nvim

Neovimの設定ファイルが、この環境を前提として書いている点にご注意ください。

# 1. 仮想環境に入ってからNeovimを起動する

仮想環境をアクティブにした後でNeovimを起動します。
特に設定をしていない状態のpyrightは、Neovim起動時に現在の```python```コマンドで呼び出す環境を認識します。
これだけで正常に仮想環境を認識します。

Ubuntuの場合

```sh
source .venv/bin/activate
```

Windowsの場合

```sh
.venv/Scripts/activate
```

その後、以下のコマンドでNeovimを起動します。

```sh
nvim
```

# 2. pyright用の設定ファイルを用意する

pyrightのドキュメント[(Configuration)](https://microsoft.github.io/pyright/#/configuration)によると、pyrightが読み込む設定ファイルとして以下の2種類があります。

- pyrightconfig.json
- pyproject.toml

どちらかのファイルをプロジェクトのルートフォルダに作成してください。
それぞれの記述例を以下に示します。ちなみに例はプロジェクトルートから見た相対パスを記述しています。(nvim-lspconfigがプロジェクトルートを探してくれる)

### pyrightconfig.json

設定項目として、venvPathに仮想環境のディレクトリがあるパス、venvに仮想環境のディレクトリ名を記述します。

```json
{
  "venvPath": ".",
  "venv": ".venv"
}
```

### pyproject.toml

pyrightconfig.jsonと同様の内容を記述します。uvでプロジェクト管理している場合、すでにpyproject.tomlが存在しているため、以下を追記するだけで済みます。

```toml
[tool.pyright]
venvPath = "."
venv = ".venv"
```

ついでに型チェックの設定なども含めれば、再利用性が高まります。

# 3. nvim-lspconfigの設定でpythonPathを指定する

サーバーのセッティングで```pythonPath```を指定するという方法もあります。ドキュメントの[Language Server Settings](https://microsoft.github.io/pyright/#/settings)に設定項目があります。
こちらはNeovimの設定ファイル内のnvim-lspconfigの設定で直接指定できます。

以下はUbuntuを例にした記述例です。

```lua
require"lspconfig".pyright.setup{
  settings = {
    python = {
      pythonPath = '.venv/bin/python', -- Windowsなら .venv/Scripts/python.exe
    }
  }
}
```

私は[kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim)を参考にしているため、init.lua内のservers変数に以下のように追記しています。

```lua
local servers = {
  pyright = {
    settings = {
      python = { pythonPath = '.venv/bin/python' },
    },
  },
  -- 以下はその他のサーバーの設定
}
```

# 終わりに

私はプロジェクト毎にファイルを用意するのが面倒なので、3の方法を採用しています。LSPによる補完がスムーズに動作し、すごく便利です。

あまり情報がまとまったサイトが見つからなくて困ったので、同じような方の助けになれば幸いです。
