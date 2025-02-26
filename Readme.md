# Visual.nvim

**N.B. This is still in a very experimental stage and mappings will
suddenly change in the next few weeks.** 

### Serendipity

noun,  formal; _the fact of finding interesting or valuable things by chance_

https://github.com/00sapo/visual.nvim/assets/22996003/804655f4-b731-4145-b765-025a5917c563

## What is this
In nvim, you do `c3w`. Ah no! It was wrong, let's retry: `uc5w`... wooops! Sorry, it
is still wrong: `uc6w`!

In Kakoune (which inspired Helix), you do the opposite: `3w`. First select 3
words, then you see you still need three words, so `3w`. Then finally `d` for
deleting.

In `visual.nvim`, this actually becomes `3wv3wd`, with the `v` used for
"refining" selections. If you do not need to adjust the selection, `3wd` is all
you need. The magic here is that `visual.nvim` puts you in a special mode named
"serendipity" in which you can use some normal commands but also some visual commands.
This allows you to have a preview of what your edit command will modify, so that you
can occasionally change the selection by entering the visual mode.

First select, then edit. This should be the way.

If you have been tempted by Kakoune and Helix editors, this may be your new plugin!

## Features

* Selection first mode
* New "serendipity" mode: discover motion errors by chance!
* Surrounding commands (change, delete, add) that operate over selections
* Repeat motions
* Compatible with treesitter-textobjects
* Compatible with leap.nvim and flit.nvim (and most of similar plugins)

## Usage

Just install it using your preferred package manager (Lazy recommended).

```lua
{ 
    '00sapo/visual.nvim',
    event = "VeryLazy", -- this is for making sure our keymaps are applied after the others: we call the previous mapppings, but other plugins/configs usually not!
}
```

Further configuration examples [below](#example-config). The mappings can be fine-tuned at your will as well,
see [Keymaps](#keymaps)

### Usage

Motion commands such as `w`, `e`, `b`, `ge`, `f`, `t` and their punctuation-aware alternatives
`W`, `E`, `B`, `gE`, `F`, `T` behave the same as vim (or are supposed so), but also put you in
"serendipity" mode.

Once you are in serendipity mode, you can modify text (`c`, `x`, `i`, `a`) as if you were in normal mode. `d` and `y` will work as in visual mode. With `<A-,>`, you can repeat the last motion selection, while with `<A-.>` you can repeat the last edit applied in insert mode from serendipity or visual mode.

Remember using `o` to move the cursor to the other end of the visual/serendipity
selection when needed. You can also use `<A-o` from serendipity mode to extend the selection to the previous cursor position. Together with `h`, `j`, `k`, `l`, these keys are a nice way for adjusting the selection in serendipity mode.

Serendipity mode is built around the nvim's visual mode, so you can use all
visual commands if they don't interfer with your config.
From serendipity mode, you can enter visual mode with `v`, `<S-v>`, `<C-v>`. You can
also switch between visual and serendipity mode with `-`. Moreover, `d` from normal mode
will be the same as `<S-v>`, followed by serendipity enter `-`.
From serendipity mode, `<esc>` will lead you to normal mode.

Note that motion commands in visual mode are different from normal mode.
Serendipity mode emulates normal mode for motion commands!

Selection of text objects is possible as in usual nvim with `i<text object>` and `a<text object>` in visual mode, thus becoming `va` and `vi` from normal mode, similarly to Helix's `mi` and `ma`. From serendipity mode, since `i` and `a` are mapped to `append` and `insert`, they become `I` and `A`, (or still `vi` and `va`). If you are a treesitter-textobjects user, simply set the `treesitter_textobjects` option to `true` for using them from serendipity mode.

Visual.nvim also offers surrounding commands with `sd`, `sc`, and `sa` (delete, change, add).

Finally, in serendipity mode, pressing `hjkl` will extend the selection. You can move to
normal mode with `<A-h>`, `<A-j>`, etc.

If you install leap.nvim and flit.nvim, you can use `s/S/f/F/t/T` for "smart" jumps. For other simila plugins (e.g. hop.nvim, flash.nvim, the `f/F/t/T` keys should work out-of-the-box, while the `s/S` keys require the option `s_jumps = true`.

### Limitations

* Visual.nvim still does not support macros due to limitations in the lua API ([see
issue](https://github.com/00sapo/visual.nvim/issues/7)). For this reason, pressing `q` will disable
Visual.nvim mappings. You should re-enable them manually with `:VisualEnable` when you
have finished recording/running the macros.

* Dot-repeat are supported via `A-.` and `A-,`. Moreover, usual `.` works from normal
mode. This may create confusion in the workflow.

* The way movements work is still being fine-tuned. Help wanted!


### Example config

Configuration with some changes to commands in order to make them compatible (needed by
NvChad):
```lua
{
  '00sapo/visual.nvim',
  config = function()
    require('visual').setup({
    commands = {
      move_up_then_normal = { amend = true },
      move_down_then_normal = { amend = true },
      move_right_then_normal = { amend = true },
      move_left_then_normal = { amend = true },
    },
  } )
  end,
  event = "VeryLazy"
}
```

Configuration for dark color scheme (changing the serendipity color, see [here](https://web.archive.org/web/20230321113552/https://codeyarns.com/tech/2011-07-29-vim-chart-of-color-names.html) for a list of (n)vim colors):
```lua
{
  '00sapo/visual.nvim',
  config = function()
    require('visual').setup({
    serendipity = {
        highlight = "guibg=LightCyan guifg=none"
    }
  } )
  end,
  event = "VeryLazy"
}
```

Example with Treesitter text objects
```lua
{
  "00sapo/visual.nvim",
  opts = { treesitter_textobjects = true },
  dependencies = { "nvim-treesitter", "nvim-treesitter-textobjects" }, -- this is needed so that visual.nvim is loaded *afterwards* Treesitter
  event = "VeryLazy"
},
{
  "nvim-treesitter/nvim-treesitter",
  --etc.
}
```

Example with custom mappings (more info [below](#keymaps))
```lua
{
  '00sapo/visual.nvim',
  opts = {
    mappings = {
      save = "<C-s>",
      docs = "K",
      to_normal = "jk"
    },
    commands = {
      save = {
        pre_amend = { "<sde>", "<esc>", "<cmd>w<cr>" }, -- <sde> is for exiting serendipity, also <sdi> for init it and <sdt> for toggling
        post_amend = {},
        modes = { "sd", "v" }, -- "sd", "v", or "n"
        amend = false, -- if true, also run the original keymap
        countable = false, -- can this mapping be counted (e.g. 3w, 3e, etc.)
      },
      docs = {
        pre_amend = {
          "<sde><esc>",
          vim.lsp.buf.hover
        },
        post_amend = {},
        modes = { "sd" },
        amend = false, -- can't use true, because keys feeded to nvim are the mode seen by `K` is visual even after <esc>. Same issue as macros.
        countable = false,
      },
      to_normal = { -- only `amend` and `countable` keys are needed and only if true
        {"<sde><esc>"}, {}, {"sd", "n"}
      }
    },
  }
  event = "VeryLazy"
}
```


## Keymaps

The plugin is highly customizable. It maps commands to keymaps, and you can define new
commands or edit the existing ones.

Feel free to suggest new default keybindings in the issues!

See the [full default options](https://github.com/00sapo/visual.nvim/blob/main/lua/visual/defaults.lua) with documentation in the comments.

# Testing

* `curl https://raw.githubusercontent.com/00sapo/visual.nvim/main/test/init.lua -o /tmp/visual.nvim-test.lua`
* `nvim -u /tmp/visual.nvim-test.lua <file>`

# Developing

* Use `nvim -u test/init.lua test/init.lua` for testing
* Use `Vdbg(...)` from anywhere to debug lua objects
* Use `<A-d>u` to show the log of debugged objects and `<A-d>c` to clean the log
