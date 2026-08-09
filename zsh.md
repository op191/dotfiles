插件下载

```bash
sudo pacman -S zsh-autosuggestions fast-syntax-highlighting zsh-completions
```

zsh配置代码

```bash
# ===== 历史记录 =====
HISTSIZE=50000              # 内存里保留的历史命令条数
SAVEHIST=50000               # 写入 ~/.zsh_history 文件的条数
HISTFILE=~/.zsh_history
setopt SHARE_HISTORY         # 多个终端窗口共享历史
setopt HIST_IGNORE_ALL_DUPS  # 整个历史里的重复命令都去重(不只是连续重复)
setopt HIST_REDUCE_BLANKS    # 去掉命令里多余的空格再记录
setopt HIST_VERIFY           # 用 !! !$ 等历史扩展时先显示出来，回车确认再执行，防止手滑执行错命令
setopt EXTENDED_HISTORY      # 历史记录里额外保存时间戳

# ===== 自动补全建议(zsh-autosuggestions) =====
# 补全建议配置(必须在 source 之前)
ZSH_AUTOSUGGEST_STRATEGY=(history completion)  # 先按历史匹配，没有再按当前可补全项匹配
#ZSH_AUTOSUGGEST_BUFFER_MAX_SIZE=20
source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh

# ===== 补全系统 =====
# 额外补全定义(需要加到 fpath,再 compinit)
fpath+=(/usr/share/zsh/plugins/zsh-completions/)
autoload -Uz compinit
# 一天内已经校验过的话直接读缓存，不用每次开终端都全量扫描 fpath,加快启动速度
if [[ -n ${ZDOTDIR:-$HOME}/.zcompdump(#qN.mh+24) ]]; then
    compinit
else
    compinit -C
fi
# 大小写补全自动转换
zstyle ':completion:*' matcher-list 'm:{a-zA-Z}={A-Za-z}'
# 补全菜单可以用方向键选择，而不是一直按 Tab 循环
zstyle ':completion:*' menu select
# 补全列表按 LS_COLORS 上色，跟 ls 输出风格一致
zstyle ':completion:*' list-colors "${(s.:.)LS_COLORS}"

# ===== Git 提示符信息 =====
autoload -Uz vcs_info
precmd() { vcs_info }
zstyle ':vcs_info:git:*' check-for-changes false
zstyle ':vcs_info:git:*' stagedstr '%F{green}✓%f'
zstyle ':vcs_info:git:*' unstagedstr '%F{red}✗%f'
zstyle ':vcs_info:git:*' formats ' (%b%c%u)'
zstyle ':vcs_info:*' enable git

# ===== 提示符样式 =====
setopt PROMPT_SUBST
PROMPT='[%F{green}%n@%m%f %F{cyan}%~%f%F{yellow}${vcs_info_msg_0_}%f]%(!.#.$) '

# ===== 其他 shell 行为 =====
setopt interactive_comments  # 交互模式下 # 后面的内容不执行，方便临时注释命令
setopt EXTENDED_GLOB         # 支持更强的通配符匹配(比如 ^、~ 排除模式)

# 文件名染色
alias ls="ls --color=auto"

# ===== 语法高亮(fast-syntax-highlighting) =====
# 语法高亮配置(必须在 source 之前)
#ZSH_HIGHLIGHT_MAXLENTH=80
source /usr/share/zsh/plugins/fast-syntax-highlighting/fast-syntax-highlighting.plugin.zsh
FAST_HIGHLIGHT_STYLES[unknown-token]='none'  # 未识别命令不再红色加粗

# ===== 快捷键 =====
bindkey '^[[1;5C' forward-word     # Ctrl+右，按单词跳转
bindkey '^[[1;5D' backward-word    # Ctrl+左，按单词跳转

# ===== 历史搜索/粘贴时取消背景高亮 =====
zle_highlight=('isearch:fg=none,bg=none' 'suffix:fg=none,bg=none' 'paste:fg=none,bg=none')

```
