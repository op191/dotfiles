在 zshrc 里加上 

```
fpath=(~/ <补全文件路径> $fpath)
autoload -Uz compinit
```

##### 如果有之前有autoload -Uz compinit，

##### 直接把fpath加在autoload -Uz compinit上方就行了

##### 一定要fpath在autoload -Uz compinit上方



```bash
#compdef gobuster

_gobuster() {
  local curcontext="$curcontext" state line
  typeset -A opt_args

  _arguments -C \
    '1:command:->cmds' \
    '*::arg:->args'

  case $state in
    cmds)
      local -a cmds
      cmds=( ${(f)"$(gobuster -h 2>/dev/null | awk '
        /^COMMANDS:/ { f=1; next }
        /^[A-Z ]+:$/ { f=0 }
        f && NF {
          name=$1; start=2
          if (name ~ /,$/) { sub(/,$/, "", name); start=3 }
          desc=""
          for (i=start; i<=NF; i++) desc = desc (i>start ? " " : "") $i
          print name ":" desc
        }')"} )
      _describe -t commands 'command' cmds
      ;;

    args)
      local -a specs fl
      local entry flags rest hasval rep desc excl pre sfx f vs

      for entry in ${(f)"$(gobuster $words[1] -h 2>/dev/null | awk '
        /^[[:space:]]+-/ {
          line=$0
          sub(/^[[:space:]]+/, "", line)
          if (match(line, /  +/)) { left=substr(line,1,RSTART-1); desc=substr(line,RSTART+RLENGTH) }
          else { left=line; desc="" }
          rep=0
          if (left ~ / \[ .* \]$/) { rep=1; sub(/ \[ .* \]$/, "", left) }
          n=split(left, parts, /, /)
          flags=""; hasval=0
          for (i=1; i<=n; i++) {
            split(parts[i], w, " ")
            if (w[1] !~ /^-/) continue
            flags = flags (flags=="" ? "" : ",") w[1]
            if (w[2] != "") hasval=1
          }
          if (flags=="" || flags ~ /(^|,)(-h|--help)(,|$)/) next
          gsub(/\[/, "\\[", desc); gsub(/\]/, "\\]", desc); gsub(/:/, "\\:", desc)
          printf "%s\t%d\t%d\t%s\n", flags, hasval, rep, desc
        }')"}; do
        flags=${entry%%$'\t'*};  rest=${entry#*$'\t'}
        hasval=${rest%%$'\t'*};  rest=${rest#*$'\t'}
        rep=${rest%%$'\t'*};     desc=${rest#*$'\t'}
        fl=( ${(s:,:)flags} )

        if (( rep )); then pre='*'; excl=''; else pre=''; excl="(${(j: :)fl})"; fi

        vs=''
        (( hasval )) && vs=':value:_files'

        for f in $fl; do
          sfx=''
          (( hasval )) && [[ $f == --* ]] && sfx='=-'
          specs+=("${excl}${pre}${f}${sfx}[${desc}]${vs}")
        done
      done

      _arguments -S $specs
      ;;
  esac
}

_gobuster "$@"

```
