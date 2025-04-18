Skip to content
# Leptos
Leptos is a Rust based web framework. You can read more about Leptos on their official website. This guide is accurate as of Leptos version 0.6.
## Checklist
  * Use SSG, Tauri doesn’t officially support server based solutions.
  * Use `serve.ws_protocol = "ws"` so that the hot-reload websocket can connect properly for mobile development.
  * Enable `withGlobalTauri` to ensure that Tauri APIs are available in the `window.__TAURI__` variable and can be imported using `wasm-bindgen`.


## Example Configuration
  1. ##### Update Tauri configuration
src-tauri/tauri.conf.json```

{
"build": {
"beforeDevCommand": "trunk serve",
"devUrl": "http://localhost:1420",
"beforeBuildCommand": "trunk build",
"frontendDist": "../dist"
},
"app": {
"withGlobalTauri": true
}
}

```

  2. ##### Update Trunk configuration
Trunk.toml```

[build]
target = "./index.html"
[watch]
ignore = ["./src-tauri"]
[serve]
port = 1420
open = false
ws_protocol = "ws"

```



© 2025 Tauri Contributors. CC-BY / MIT
