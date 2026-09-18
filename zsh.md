插件下载

```bash
sudo pacman -S zsh-syntax-highlighting zsh-autosuggestions
```

zsh配置代码

```bash
# 使用方法：将此文件内容覆盖到 ~/.zshrc，然后执行 `source ~/.zshrc` 或重开终端

# ===== 基础 shell 选项 =====
setopt autocd              # 直接输入目录名即可 cd 进去
setopt interactivecomments # 交互模式下允许写 # 注释
setopt magicequalsubst     # 对 foo=bar 形式的参数也做文件名展开
setopt nonomatch           # 通配符匹配不到时不报错
setopt notify              # 后台任务状态变化立即通知
setopt numericglobsort     # 文件名里的数字按数值大小排序，而非字符串排序
setopt promptsubst         # 允许在 PROMPT 里做命令替换

WORDCHARS='_-' # 编辑命令行时，_ 和 - 不算作单词分隔符（ctrl+方向键跳词用）

PROMPT_EOL_MARK="" # 隐藏输出末尾没有换行符时显示的 % 符号

# ===== 按键绑定 =====
bindkey -e                                        # emacs 风格快捷键（ctrl+a/e/w 等）
bindkey ' ' magic-space                           # 空格键触发历史展开（如 !!<space>）
bindkey '^U' backward-kill-line                   # ctrl+U 删除光标前整行
bindkey '^[[3;5~' kill-word                       # ctrl+Delete
bindkey '^[[3~' delete-char                       # Delete 键
bindkey '^[[1;5C' forward-word                    # ctrl+右方向键 跳词
bindkey '^[[1;5D' backward-word                   # ctrl+左方向键 跳词
bindkey '^[[5~' beginning-of-buffer-or-history    # Page Up
bindkey '^[[6~' end-of-buffer-or-history          # Page Down
bindkey '^[[H' beginning-of-line                  # Home
bindkey '^[[F' end-of-line                        # End
bindkey '^[[Z' undo                               # shift+Tab 撤销上一步操作

# ===== 补全系统 =====
autoload -Uz compinit
compinit -d ~/.cache/zcompdump   # 缓存补全数据库，避免每次启动重新扫描
zstyle ':completion:*:*:*:*:*' menu select
zstyle ':completion:*' auto-description 'specify: %d'
zstyle ':completion:*' completer _expand _complete
zstyle ':completion:*' format 'Completing %d'
zstyle ':completion:*' group-name ''
zstyle ':completion:*' list-colors ''
zstyle ':completion:*' list-prompt %SAt %p: Hit TAB for more, or the character to insert%s
zstyle ':completion:*' matcher-list 'm:{a-zA-Z}={A-Za-z}'  # 补全时大小写不敏感
zstyle ':completion:*' rehash true
zstyle ':completion:*' select-prompt %SScrolling active: current selection at %p%s
zstyle ':completion:*' use-compctl false
zstyle ':completion:*' verbose true
zstyle ':completion:*:kill:*' command 'ps -u $USER -o pid,%cpu,tty,cputime,cmd'  # kill 命令补全时显示进程列表

# ===== 历史记录 =====
HISTFILE=~/.zsh_history
HISTSIZE=1000
SAVEHIST=2000
setopt hist_expire_dups_first # 历史记录超限时优先删除重复命令
setopt hist_ignore_dups       # 连续重复的命令不重复记录
setopt hist_ignore_space      # 以空格开头的命令不记录（临时敏感命令可用这招）
setopt hist_verify            # 历史展开（如 !123）先显示命令，回车确认后再执行
#setopt share_history         # 多终端间实时共享历史，默认关闭

alias history="history 0"     # 显示完整历史（不截断编号）

TIMEFMT=$'\nreal\t%E\nuser\t%U\nsys\t%S\ncpu\t%P'  # time 命令输出格式

# ===== 提示符（prompt）=====
# ===== 提示符（prompt）=====
if [ -x /usr/bin/tput ] && tput setaf 1 >&/dev/null; then
    color_prompt=yes
else
    color_prompt=
fi

# ===== git 分支信息（供下面 PROMPT 里的 vcs_info_msg_0_ 使用）=====
autoload -Uz vcs_info
zstyle ':vcs_info:git:*' formats ' (%b)'   # 显示格式：空格 + 括号包裹的分支名
precmd_functions+=(vcs_info)               # 每次显示提示符前刷新一次 git 状态

#NEWLINE_BEFORE_PROMPT=yes    # 每次输出后空一行再显示新提示符

if [ "$color_prompt" = yes ]; then
    VIRTUAL_ENV_DISABLE_PROMPT=1

    # ===== 自定义单行提示符：SSH 连接时用户名@主机名变青色，本地是绿色 =====
    PROMPT='%B%F{${${SSH_CONNECTION:+cyan}:-green}}%n@%m%f%b:%B%F{blue}%~%f%b%F{yellow}${vcs_info_msg_0_}%f%(!.#.$) '

    # ===== ssh 命令包装：主动连出去时整个终端变色，提醒自己身处远程会话 =====
    ssh() {
        {
            echo -ne '\e[1;36m'   # 连接前，终端切成加粗青色
            command ssh "$@"      # command 避免递归调用自己
        } always {
            echo -ne '\e[0m'      # 正常退出/断线/Ctrl+C 都强制重置回默认色
        }
    }

    # ===== 语法高亮插件 =====
    if [ -f /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh ]; then
        . /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
        ZSH_HIGHLIGHT_HIGHLIGHTERS=(main brackets pattern)
        ZSH_HIGHLIGHT_STYLES[default]=none
        ZSH_HIGHLIGHT_STYLES[unknown-token]=underline
        ZSH_HIGHLIGHT_STYLES[reserved-word]=fg=cyan,bold
        ZSH_HIGHLIGHT_STYLES[suffix-alias]=fg=green,underline
        ZSH_HIGHLIGHT_STYLES[global-alias]=fg=green,bold
        ZSH_HIGHLIGHT_STYLES[precommand]=fg=green,underline
        ZSH_HIGHLIGHT_STYLES[commandseparator]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[autodirectory]=fg=green,underline
        ZSH_HIGHLIGHT_STYLES[path]=bold
        ZSH_HIGHLIGHT_STYLES[path_pathseparator]=
        ZSH_HIGHLIGHT_STYLES[path_prefix_pathseparator]=
        ZSH_HIGHLIGHT_STYLES[globbing]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[history-expansion]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[command-substitution]=none
        ZSH_HIGHLIGHT_STYLES[command-substitution-delimiter]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[process-substitution]=none
        ZSH_HIGHLIGHT_STYLES[process-substitution-delimiter]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[single-hyphen-option]=fg=green
        ZSH_HIGHLIGHT_STYLES[double-hyphen-option]=fg=green
        ZSH_HIGHLIGHT_STYLES[back-quoted-argument]=none
        ZSH_HIGHLIGHT_STYLES[back-quoted-argument-delimiter]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[single-quoted-argument]=fg=yellow
        ZSH_HIGHLIGHT_STYLES[double-quoted-argument]=fg=yellow
        ZSH_HIGHLIGHT_STYLES[dollar-quoted-argument]=fg=yellow
        ZSH_HIGHLIGHT_STYLES[rc-quote]=fg=magenta
        ZSH_HIGHLIGHT_STYLES[dollar-double-quoted-argument]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[back-double-quoted-argument]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[back-dollar-quoted-argument]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[assign]=none
        ZSH_HIGHLIGHT_STYLES[redirection]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[comment]=fg=black,bold
        ZSH_HIGHLIGHT_STYLES[named-fd]=none
        ZSH_HIGHLIGHT_STYLES[numeric-fd]=none
        ZSH_HIGHLIGHT_STYLES[arg0]=fg=cyan
        ZSH_HIGHLIGHT_STYLES[bracket-error]=fg=red,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-1]=fg=blue,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-2]=fg=green,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-3]=fg=magenta,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-4]=fg=yellow,bold
        ZSH_HIGHLIGHT_STYLES[bracket-level-5]=fg=cyan,bold
        ZSH_HIGHLIGHT_STYLES[cursor-matchingbracket]=standout
    fi
else
    PROMPT='%n@%m:%~%(#.#.$) '
fi
unset color_prompt force_color_prompt

# xterm 系终端下，把窗口标题设成 用户@主机:路径
case "$TERM" in
xterm*|rxvt*|Eterm|aterm|kterm|gnome*|alacritty)
    TERM_TITLE=$'\e]0;%n@%m: %~\a'
    ;;
*)
    ;;
esac

precmd() {
    print -Pnr -- "$TERM_TITLE"
    # 每次显示新提示符前空一行（第一次不空，避免开屏就有空行）
    if [ "$NEWLINE_BEFORE_PROMPT" = yes ]; then
        if [ -z "$_NEW_LINE_BEFORE_PROMPT" ]; then
            _NEW_LINE_BEFORE_PROMPT=1
        else
            print ""
        fi
    fi
}

# ===== ls/grep 等命令上色 + 常用别名 =====
if [ -x /usr/bin/dircolors ]; then
    test -r ~/.dircolors && eval "$(dircolors -b ~/.dircolors)" || eval "$(dircolors -b)"
    export LS_COLORS="$LS_COLORS:ow=30;44:" # 修正 777 权限目录的显示颜色

    alias ls='ls --color=auto'
    alias grep='grep --color=auto'
    alias fgrep='fgrep --color=auto'
    alias egrep='egrep --color=auto'
    alias diff='diff --color=auto'
    alias ip='ip --color=auto'

    # less/man 里的高亮颜色（终端控制码）
    export LESS_TERMCAP_mb=$'\E[1;31m'
    export LESS_TERMCAP_md=$'\E[1;36m'
    export LESS_TERMCAP_me=$'\E[0m'
    export LESS_TERMCAP_so=$'\E[01;33m'
    export LESS_TERMCAP_se=$'\E[0m'
    export LESS_TERMCAP_us=$'\E[1;32m'
    export LESS_TERMCAP_ue=$'\E[0m'
    export MANROFFOPT="-c"

    zstyle ':completion:*' list-colors "${(s.:.)LS_COLORS}"
    zstyle ':completion:*:*:kill:*:processes' list-colors '=(#b) #([0-9]#)*=0=01;31'
fi

alias ll='ls -l'
alias la='ls -A'
alias l='ls -CF'

# ===== 自动建议插件 =====

if [ -f /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh ]; then
    . /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
    ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE='fg=244'   # 建议文字的颜色（灰色）
fi

# 注：Kali 原版这里还有 command-not-found 钩子（/etc/zsh_command_not_found）


```
