插件下载

```bash
sudo pacman -S zsh-autosuggestions fast-syntax-highlighting zsh-completions
```

zsh配置代码

```bash
HISTSIZE=1000        # 内存里保留的历史命令条数
SAVEHIST=1000         # 写入 ~/.zsh_history 文件的条数
HISTFILE=~/.zsh_history
setopt SHARE_HISTORY      # 多个终端窗口共享历史(可选)
setopt HIST_IGNORE_DUPS   # 连续重复命令不重复记录(可选,减少历史膨胀)


# 补全建议配置(必须在 source 之前)
#SH_AUTOSUGGEST_STRATEGY=(history)
#ZSH_AUTOSUGGEST_BUFFER_MAX_SIZE=20
source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh

# 额外补全定义(需要加到 fpath,再 compinit)
fpath+=(/usr/share/zsh/plugins/zsh-completions/)

# 初始化补全系统
autoload -Uz compinit
compinit

# 大小写补全自动转换
zstyle ':completion:*' matcher-list 'm:{a-zA-Z}={A-Za-z}'

# git 提示符信息
autoload -Uz vcs_info
precmd() { vcs_info }
zstyle ':vcs_info:git:*' check-for-changes true
zstyle ':vcs_info:git:*' stagedstr '%F{green}✓%f'
zstyle ':vcs_info:git:*' unstagedstr '%F{red}✗%f'
zstyle ':vcs_info:git:*' formats ' (%b%c%u)'
zstyle ':vcs_info:*' enable git

# 提示符样式
setopt PROMPT_SUBST
PROMPT='[%F{green}%n@%m%f %F{cyan}%~%f%F{yellow}${vcs_info_msg_0_}%f]%(!.#.$) '

# 不执行注释#后的内容
setopt interactive_comments

# 文件名字染色
alias ls="ls --color=auto"

# 语法高亮配置(必须在 source 之前)
#ZSH_HIGHLIGHT_MAXLENGTH=80
source /usr/share/zsh/plugins/fast-syntax-highlighting/fast-syntax-highlighting.plugin.zsh

```
