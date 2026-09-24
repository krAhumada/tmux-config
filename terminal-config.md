# Configuración de Terminal + Tmux

> Fecha: 17/09/2026 · Shell: bash · Tmux 3.2a · Starship 1.26.0

---

## 1. Instalación en la máquina nueva (en orden)

```bash
# 1. Dependencias base
sudo apt update
sudo apt install -y tmux xclip

# 2. Starship (prompt)
curl -sS https://starship.rs/install.sh | sh

# 3. Tomar la sección "~/.tmux.conf" que esta más abajo
mv EXPORTAR_CONFIG_TERMINAL.md ~/.tmux.conf   # o copiar manualmente

# 4. Instalar TPM (gestor de plugins)
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# 5. Bash: agregar starship + alias al final de ~/.bashrc
echo 'eval "$(starship init bash)"' >> ~/.bashrc
echo 'alias lzd='"'"'lazydocker'"'"'' >> ~/.bashrc
source ~/.bashrc
```

> **Importante:** al completar, copiar también la sección de `~/.bashrc` (paso 2) y
> `starship.toml` (paso 3) según corresponda.

```bash
# 6. Dentro de tmux: instalar catppuccin
tmux new -s setup
#   presionar: C-a + I   (esperar a que instale)
#   verificar con:       C-a + :  →  source ~/.tmux.conf
# Terminado, probar el layout 60/40 con Enter
```

**Verificar:** dentro de tmux ejecutar `echo $TERM` debe dar `tmux-256color`.

---

## 2. `~/.tmux.conf`

```tmux
# --- 1. Comportamiento (Lo que importa) ---
set -g default-terminal "tmux-256color"
set -ag terminal-overrides ",xterm-256color:RGB"
set -as terminal-features ",xterm-256color:RGB"

set -g mouse on               # Mouse para scroll y redimensionar
set -s escape-time 0          # Sin lag para Neovim
set -g base-index 1           # Ventanas empiezan en 1
setw -g pane-base-index 1


# Cambiar el Prefix a Ctrl+a (más ergonómico que Ctrl+b)
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# Dividir paneles con teclas lógicas ( - y | )
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# cambiar el panel pequeño al panel principal con enter
set-window-option -g main-pane-width 60%
bind-key Enter swap-pane -t :.1 \; select-layout main-vertical \; select-pane -t :.1

# Moverse entre paneles como en Vim (¡clave para LazyVim!)
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# muestra el nombre del panel
set -g pane-border-status top
set -g pane-border-format " [ #P: #{b:pane_current_path}#{?#{==:#{pane_current_command},bash},, | #T} ] "
set -g focus-events on

# --- Navegación Instantánea (Alt + Número) ---
# El parámetro -n significa que NO necesitas presionar el prefijo (Ctrl-a)
bind-key -n M-1 select-pane -t 1
bind-key -n M-2 select-pane -t 2
bind-key -n M-3 select-pane -t 3
bind-key -n M-4 select-pane -t 4
bind-key -n M-5 select-pane -t 5
bind-key -n M-6 select-pane -t 6
bind-key -n M-7 select-pane -t 7
bind-key -n M-8 select-pane -t 8
bind-key -n M-9 select-pane -t 9

bind -n F2 display-popup -w 75% -h 75% -E $SHELL

# Usar teclas de Vim en el modo copia
set -g mode-keys vi

# Mantener la selección visual al soltar el mouse (evita que se quite la selección)
unbind -T copy-mode-vi MouseDragEnd1Pane

# Copiar al portapapeles del sistema al presionar 'y' o al terminar de arrastrar
# Requiere tener instalado 'xclip' o 'xsel'
bind-key -T copy-mode-vi v send-keys -X begin-selection
bind-key -T copy-mode-vi y send-keys -X copy-pipe "xclip -selection clipboard -i"

# Opción para que al soltar el mouse también copie al sistema
bind-key -T copy-mode-vi MouseDragEnd1Pane send-keys -X copy-pipe "xclip -selection clipboard -i"

# Presiona Prefix + y para activar/desactivar la sincronización
bind-key y set-window-option synchronize-panes

# Atajo para renombrar el panel actual (Prefix + T)
bind-key T command-prompt -p "Nombre del panel:" "select-pane -T '%%'"

# ==========================================
# --- CONFIGURACIÓN DE APARIENCIA (catppuccin) ---
# ==========================================
set -g @plugin 'tmux-plugins/tpm'
#set -g @plugin "arcticicestudio/nord-tmux"     # (no en uso, eliminado)
set -g @plugin 'catppuccin/tmux'

# Catppuccin settings
set -g @catppuccin_flavor 'mocha'  # Oscuro y moderno
set -g @catppuccin_window_status_style "rounded"
set -g @catppuccin_window_current_text "#W"
set -g @catppuccin_window_default_text "#W"

# Pane borders más visibles
set -g pane-border-style "fg=#585b70"
set -g pane-active-border-style "fg=#a6e3a1"  # Color Catppuccin
set -g window-active-style "bg=#2E3440"

# 2. BLOQUEO TOTAL DE RENOMBRADO AUTOMÁTICO
set-option -g allow-rename off
set-window-option -g automatic-rename off

# --- EL TRUCO MAESTRO DE CONTROL TOTAL ---
set-hook -g window-linked 'set-window-option automatic-rename off'
set-hook -g after-select-window 'set-window-option automatic-rename off'

# Inicializar el gestor de plugins (debe estar al final del archivo)
run '~/.tmux/plugins/tpm/tpm'

# (catppuccin se auto-ejecuta con TPM → `run catppuccin.tmux` redundante, eliminado)

set -g window-status-format "#[fg=#11111b,bg=#{@thm_overlay_2}]#[fg=#181825,reverse]#[none]#I #[fg=#cdd6f4,bg=#{@thm_surface_0}]#W#[fg=#181825,reverse]#[none]"
set -g status-right-length 150
set -g status-left "#{E:@catppuccin_status_session}"
set -g status-right "#{E:@catppuccin_status_directory}#{E:@catppuccin_status_date_time}"
```

---

## 3. `~/.config/starship.toml`

```toml
# Starship - Configuración Simple y Funcional

format = "$all$directory$git_branch$git_status$time$line_break$character"

[character]
success_symbol = "[❯](bold green)"
error_symbol = "[❯](bold red)"

[directory]
style = "bold blue"
truncation_length = 3
truncation_symbol = "…/"

[git_branch]
symbol = " "
style = "bold magenta"

[git_status]
style = "bold yellow"

[time]
disabled = false
style = "bold white"
format = "[$time]($style) "
```

---

## 4. Bloques de `~/.bashrc` (terminal/tmux)

```bash
export PATH="$PATH:/opt/nvim-linux-x86_64/bin"

export LC_ALL=C.UTF-8

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"                   # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion" # This loads nvm bash_completion

# opencode
export PATH=/home/ubuntu2204/.opencode/bin:$PATH

# STARTSHIP PROMPT
eval "$(starship init bash)"
alias lzd='lazydocker'
```

---

## 5. Notas / dependencias opcionales

- **Nvim/LazyVim** para integrarse con tmux (uso de `h/j/k/l`, escape-time 0).
- **xclip** → apt: `sudo apt install -y xclip`.
- Los plugins `tpm` y `catppuccin/tmux` se **reinstalan** en la máquina nueva vía TPM (`C-a + I`), no se exportan.
