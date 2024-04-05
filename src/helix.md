# Helix Configuration

## Global
```toml
[editor]
bufferline = "multiple"
idle-timeout = 10
soft-wrap.enable = true
workspace-lsp-roots = ["migration"] # sea-orm wants migration inside another project

[keys.normal]
"'" = { d = ":buffer-close", o = ":buffer-close-others", c = ":sh alacritty --hold -e cargo clippy", b = ":sh alacritty --hold -e cargo build", r = ":sh alacritty -e cargo run", R = ":sh alacritty --hold -e cargo run", t = ":sh alacritty --hold -e cargo test" }
A-j = ":buffer-previous"
A-k = ":buffer-next"

[keys.normal.space]
i = ":toggle lsp.display-inlay-hints"
```

## Local
Place in project dir `.helix/languages.toml`

Embedded
```toml
[[language]]
name = "rust"

[language-server.rust-analyzer.config]
check.allTargets = false
cargo.target = "riscv32imac-unknown-none-elf"
```
Leptos
```toml
[[language]]
name = "rust"
[language.formatter]
command = "sh"
args = ["-c", "rustfmt --edition 2021 | leptosfmt --stdin"]
[language-server.rust-analyzer]
config = { procMacro = { ignored = { leptos_macro = [ "server" ] } } } 
```
Dioxus
```toml
[[language]]
name = "rust"
language-servers = ["rust-analyzer", "tailwindcss-ls"]

[language-server.tailwindcss-ls]
config = { userLanguages = { rust = "html", "*.rs" = "html" }, tailwindCSS.experimental.classRegex = ["class: \"(.*)\""] }
```
