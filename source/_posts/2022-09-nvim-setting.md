---
title: Neovim 設定
categories:
  - Technical
tags:
  - Neovim
  - Editors
  - LSP
date: 2022-09-21 07:51:09
---

## 前情提要
上一篇文章的最後我提到學任何語言首先要先調教好編輯器，
這是源自於我那天還沒開始寫任何 code，就先花了半天的時間在處理 Neovim 的設定，特別是 LSP 的設定真的差點搞死我……
乾脆寫一篇來紀錄跟分享這個過程。
~~我是不是要感謝我的笨拙跟容易翻車讓我常常有題材可以寫文章~~

## 為什麼是純文字編輯器
或許有人會問：「現在新式的 IDE 像是 VS Code 界面漂亮而且使用方便，為什麼還要用純文字編輯器？」
這有很多可以說的面向，而且每個人的理由也不盡相同，
我自己的話是想要挑戰看看純文字編輯器的功能是不是真的不如 IDE、以及單純覺得好玩（o
當然專案開發我自己還是以 IDE 為主，因為視覺化的看專案檔真的比較輕鬆一點。
（BTW，雖然我前天 Java 是用 Neovim 寫的，
但強者我朋友 a.k.a. 清大怪物高秋表示「Java『專案』」是他數一數二推薦用 IDE 進行開發的語言，
因為純用 command line 會很煩躁）

## 為什麼是 Neovim
純文字編輯器主流分成兩大系譜：emacs 跟 vi，那為什麼我選擇了 vi 這支而不是 emacs 系？
~~因為 emacs 東西太多了，多到我會尖叫 too much 那種多（關於這個我不打算寫，我會逼高秋寫），~~
vi 系的好處，或者說優勢是在 terminal 上比較便利，emacs 只有 GUI 版比較好用，
如果想嘗試終端機界面 coding，用 vi 系的體驗會比較好；
至於選擇 Neovim 是因為 plug-in 的生態比較活躍，
並且自行撰寫 plug-in 也比較容易~~，這是個 plug-in 的[新時代](https://www.youtube.com/watch?v=1FliVTcX8bQ)~~。

## Neovim 設定~~調教~~
好我花太多篇幅寫為什麼了，趕快進入正題（（（

先把 Neovim 安裝好（怎麼安裝？去搜尋！），接著要來改一下設定檔囉～
### init.vim
這是 Neovim 的全域設定檔，通常應該在 `~/.config/nvim/` 之下，
然後隨便開個編輯器來改設定就好了w

一開始的 `init.vim` 應該會看到幾乎每一行開頭都是雙引號 `"`，
這是由於 Neovim 本身預設是用 vimscript 來寫相關設定，在這個語言中 `"` 是註解（就像 C 裡面的 `//` 一樣）；
不過現在主流是使用 Lua 語言來寫設定跟 plug-in ，所以該怎麼辦呢？

**那就自己寫一個 Lua 設定就好啦！**
### Lua 設定檔
首先在剛剛的 `~/.config/nvim/` 下新增一個 `lua` 目錄，
在 `lua` 裡面建立一個設定的 `.lua` 檔案（這邊取名 `setups.lua`），
然後以下是範例檔案（至於需要加上什麼請參考各 plug-in 說明文件）：
```lua
---nvim-lsp-installer
require("nvim-lsp-installer").setup {}

---nvim-cmp

local cmp = require'cmp'

cmp.setup({
  snippet = {
    -- REQUIRED - you must specify a snippet engine
    expand = function(args)
      -- vim.fn["vsnip#anonymous"](args.body) -- For `vsnip` users.
      require('luasnip').lsp_expand(args.body) -- For `luasnip` users.
      -- require('snippy').expand_snippet(args.body) -- For `snippy` users.
      -- vim.fn["UltiSnips#Anon"](args.body) -- For `ultisnips` users.
    end,
  },
  window = {
    -- completion = cmp.config.window.bordered(),
    -- documentation = cmp.config.window.bordered(),
  },
  mapping = cmp.mapping.preset.insert({
    ['<C-b>'] = cmp.mapping.scroll_docs(-4),
    ['<C-f>'] = cmp.mapping.scroll_docs(4),
    ['<C-Space>'] = cmp.mapping.complete(),
    ['<C-e>'] = cmp.mapping.abort(),
    ['<CR>'] = cmp.mapping.confirm({ select = true }), -- Accept currently selected item. Set `select` to `false` to only confirm explicitly selected items.
  }),
  sources = cmp.config.sources({
    { name = 'nvim_lsp' },
    { name = 'vsnip' }, -- For vsnip users.
    -- { name = 'luasnip' }, -- For luasnip users.
    -- { name = 'ultisnips' }, -- For ultisnips users.
    -- { name = 'snippy' }, -- For snippy users.
  }, {
    { name = 'buffer' },
  })
})

-- Set configuration for specific filetype.
cmp.setup.filetype('gitcommit', {
  sources = cmp.config.sources({
    { name = 'cmp_git' }, -- You can specify the `cmp_git` source if you were installed it.
  }, {
    { name = 'buffer' },
  })
})

-- Use buffer source for `/` (if you enabled `native_menu`, this won't work anymore).
cmp.setup.cmdline('/', {
  mapping = cmp.mapping.preset.cmdline(),
  sources = {
    { name = 'buffer' }
  }
})

-- Use cmdline & path source for ':' (if you enabled `native_menu`, this won't work anymore).
cmp.setup.cmdline(':', {
  mapping = cmp.mapping.preset.cmdline(),
  sources = cmp.config.sources({
    { name = 'path' }
  }, {
    { name = 'cmdline' }
  })
})

-- Set up lspconfig.
local capabilities = require('cmp_nvim_lsp').update_capabilities(vim.lsp.protocol.make_client_capabilities())


--- nvim-lspconfig
-- Mappings.
-- See `:help vim.diagnostic.*` for documentation on any of the below functions
local opts = { noremap=true, silent=true }
vim.keymap.set('n', '<space>e', vim.diagnostic.open_float, opts)
vim.keymap.set('n', '[d', vim.diagnostic.goto_prev, opts)
vim.keymap.set('n', ']d', vim.diagnostic.goto_next, opts)
vim.keymap.set('n', '<space>q', vim.diagnostic.setloclist, opts)

-- Use an on_attach function to only map the following keys
-- after the language server attaches to the current buffer
local on_attach = function(client, bufnr)
---- Enable completion triggered by <c-x><c-o>
--vim.api.nvim_buf_set_option(bufnr, 'omnifunc', 'v:lua.vim.lsp.omnifunc')

---- Mappings.
---- See `:help vim.lsp.*` for documentation on any of the below functions
--local bufopts = { noremap=true, silent=true, buffer=bufnr }
--vim.keymap.set('n', 'gD', vim.lsp.buf.declaration, bufopts)
--vim.keymap.set('n', 'gd', vim.lsp.buf.definition, bufopts)
--vim.keymap.set('n', 'K', vim.lsp.buf.hover, bufopts)
--vim.keymap.set('n', 'gi', vim.lsp.buf.implementation, bufopts)
--vim.keymap.set('n', '<C-k>', vim.lsp.buf.signature_help, bufopts)
--vim.keymap.set('n', '<space>wa', vim.lsp.buf.add_workspace_folder, bufopts)
--vim.keymap.set('n', '<space>wr', vim.lsp.buf.remove_workspace_folder, bufopts)
--vim.keymap.set('n', '<space>wl', function()
  --print(vim.inspect(vim.lsp.buf.list_workspace_folders()))
--end, bufopts)
--vim.keymap.set('n', '<space>D', vim.lsp.buf.type_definition, bufopts)
--vim.keymap.set('n', '<space>rn', vim.lsp.buf.rename, bufopts)
--vim.keymap.set('n', '<space>ca', vim.lsp.buf.code_action, bufopts)
--vim.keymap.set('n', 'gr', vim.lsp.buf.references, bufopts)
--vim.keymap.set('n', '<space>f', vim.lsp.buf.formatting, bufopts)
end

local lsp_flags = {
---- This is the default in Nvim 0.7+
--debounce_text_changes = 150,
}
require('lspconfig').jdtls.setup{
--on_attach = on_attach,
--flags = lsp_flags,
capabilities = capabilities,
}
```
至於這些是什麼東西？
稍後會挑重點講解。

### vim-plug 設定
接著回到 `init.vim`，我們要先安裝 plug-in 管理，
目前主流有兩種使用 plug-in 的方式：
1. packer.vim
2. vim-plug

這次我使用了 vim-plug，至於怎麼安裝 vim-plug 並且加入 plug-in ，
可以參考另一個強者我朋友 SimbaFs 寫的[這篇文章](https://ithelp.ithome.com.tw/articles/10273424)，這邊不多加贅述。

確定 vim-plug 安裝成功後，
就可以在兩個 `call plug` 中間的區塊（見前述文章）加入你要的 plug-in 了。

### LSP plug-ins
終於要來設定重點的 LSP 了，沒有 LSP 你用 Neovim 就等於用記事本而已（o）
首先在上述區塊中加入這行：
```vimscript
Plug 'neovim/nvim-lspconfig'
```
這個負責執行 LSP server，
接著如果你的 Neovim 是 0.7.0 以後的版本（現在大部份發行板的 `latest` 都過了吧），
就可以安裝 `williamboman/nvim-lsp-installer` 這個 plug-in，可以直接在指令列下 `:LspInstall` 安裝 LSP server：
```vimscript
Plug 'williamboman/nvim-lsp-installer'
```
接下來是與自動完成相關的 plug-in，前兩者建議一定要安裝，
第三個端看你 `setups.lua` 中 expand 選擇什麼，因為我選擇了 `luasnip`，所以就是安裝這個：
```vimscript
Plug 'hrsh7th/nvim-cmp'
Plug 'hrsh7th/cmp-nvim-lsp'
Plug 'L3MON4D3/LuaSnip'
```
儲存修改後，開啟 Neovim，直接下 `:PlugInstall` 指令，會根據 `init.vim` 的內容安裝，
安裝成功就可以先下 `:q` 關閉。

### 使用 setups.lua
在稍早我們新增了一個名為 `.config/nvim/lua/setups.lua` 的檔案，
接下來就是要讓 `init.vim` 使用這個檔案進行設定。

在 `call plug#end()` 下面新增一行 `lua require('setups')`，
**注意引號內要寫的是 lua 設定檔的檔名（不要照抄！）**，
儲存後可以開啟 Neovim，沒有噴 Error 就是成功了。
~~然後就不需要再碰 vimscript 了。~~

所以這個檔案到底是什麼東西？
簡單來說是把 plug-in 的設定都寫進來，讓 Neovim 自己去讀取，
大部分設定都不用改（有興趣可以去研究看看XD），這邊要特別提及的是兩個區塊：
1. `cmp.setup()` 裡面的 `expand` 部份
2. 最下面 `require` 的部份

#### expand
```lua
expand = function(args)
  -- vim.fn["vsnip#anonymous"](args.body) -- For `vsnip` users.
  require('luasnip').lsp_expand(args.body) -- For `luasnip` users.
  -- require('snippy').expand_snippet(args.body) -- For `snippy` users.
  -- vim.fn["UltiSnips#Anon"](args.body) -- For `ultisnips` users.
end,
```
這邊是放自動完成格式的 plug-in，平行的四行是讓你選擇其一（~~記得要安裝不然會像我一樣翻車~~），
至於安裝方式就是找到 GitHub repo，以 `Plug 'author/repo_name'` 的方式寫入 `init.vim` 並安裝，
其他不需要的就是註解掉（在 Lua 裡面是使用 `--`）。

#### require
```lua
require('lspconfig').jdtls.setup{
--on_attach = on_attach,
--flags = lsp_flags,
capabilities = capabilities,
}
```
這邊是設定 LSP server 的自動完成，跟上面 `expand` 都有設定才是完整的自動完成。
把 `jdtls` 的部份代換成你要的 LSP server 就可以了（至於有哪些可以用請參考官方文件）；
有複數個 server 就如法炮製複製同樣的段落。

好啦，現在應該是可以開心的當 IDE 來寫程式囉（？），
如果要檢查有沒有正常運作可以開啟原始碼檔案，指令輸入 `:LspInfo` 檢查，
沒有問題就可以舒舒服服的 coding 啦！

## 結論
如果真的很懶還是請用 IDE，第一次操作你一定會想放棄（ｏ
不過花半天設定完看到可以正常運作的時候還是會很感動的。

另外提醒如果是用 Java 的 `jdtls`，請務必注意 OpenJDK 版本要是最新的（目前是 `openjdk-17-jre`），
不然 LSP 還是不會動的喔<3（受害者辛酸發言）

如果想挑戰自我的真的可以玩看看純文字編輯器，
你會發現新大陸www

火山 / Kazan
2022.09.21