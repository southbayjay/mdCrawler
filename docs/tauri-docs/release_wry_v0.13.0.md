Skip to content
# wry@0.13.0
ReturnView on GitHub
  * Update gtk to 0.15 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Add clipboard field in WebViewAttributes. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Ignore transparency on Windows 7 to prevent application crash. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Remove clipboard property for consistency across platforms. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Enable cookie persistence on Linux if the `data_directory` is provided. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Enable objc’s exception features so they can be treated as panic message. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Add inner size method for webview. This can reflect correct size of webview on macOS. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Add “transparent” and “fullscreen” feature flags on macOS to toggle private API. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Implement WebContextImpl on mac to extend several callback lifetimes. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * The only thing that private mod shared does is re-export http mod to public, we can just pub mod http. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  *     * Fix hovering over an edge of undecorated window on Linux won’t change cursor.
  * Undecorated window can be resized using touch on Linux.
  * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Update webkit2gtk to 0.15 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Add `with_user_agent(&amp;str)` to `WebViewBuilder`. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Replace all of the `winapi` crate references with the `windows` crate, and replace `webview2` and `webview2-sys` with `webview2-com` and `webview2-com-sys` built with the `windows` crate. The replacement bindings are in the `webview2-com-sys` crate, with `pub use` in the `webview2-com` crate. They can be shared with TAO. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Fix null pointer crash on `get_content` of web resource request. This is a temporary fix. We will switch it back once upstream is updated. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Update the `windows` crate to 0.25.0, which comes with pre-built libraries. WRY and Tao can both reference the same types directly from the `windows` crate instead of sharing bindings in `webview2-com-sys`. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Update the `windows` crate to 0.29.0 and `webview2-com` to 0.9.0. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05
  * Update the `windows` crate to 0.30.0 and `webview2-com` to 0.10.0. 
    * 219d20c Merge next back to dev branch (#477) on 2022-02-05


© 2025 Tauri Contributors. CC-BY / MIT
