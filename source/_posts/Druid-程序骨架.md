---
title: Druid-程序骨架
tags:
  - Rust
  - GUI
categories:
  - Rust
  - GUI
cover: 'https://s2.loli.net/2022/10/14/ydOW3hI8DkSJsrg.jpg'
abbrlink: 58266
date: 2022-12-11 18:16:19
---

# Cargo.toml
```toml
[package]
name = "druid-gui-app"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
druid = { git = "https://github.com/linebender/druid.git" }
tracing = { version = "0.1.22" }

[[bin]]
name = "druid-gui-app"
path = "./src/main.rs"

[[bin]]
name = "example"
path = "./previews/example.rs"
```
>有两个[[bin]]，example可以用来编写一些示例用来预览效果，另外一个是真正的需要编写的程序。分别用下面的命令运行
```shell
cargo run --bin example # 运行示例程序
cargo run --bin druid-gui-app # 运行主程序
```

# main.rs
```rust
#![windows_subsystem = "windows"]

use druid::widget::prelude::*;
use druid::widget::{Align, BackgroundBrush, Button, Controller, ControllerHost, Flex, Label, Padding, TextBox};
use druid::Target::Global;
use druid::{commands as sys_cmds, AppDelegate, AppLauncher, Application, Color, Command, Data, DelegateCtx, Handled, LocalizedString, Menu, MenuItem, Target, WindowDesc, WindowHandle, WindowId, WidgetExt, lens};
use tracing::{
    info, warn, error,
};

///可以定义多个这样的结构体，用来自定义数据和组件进行一个“绑定”
///把这个结构体和组件的data变量进行一个关联
#[derive(Data, Clone)]
struct AppData {
    data: String,
}

///程序入口函数
pub fn main() {
    // window 绑定菜单栏
    // let window = WindowDesc::new(ui_builder()).menu(make_menu);
    let window = WindowDesc::new(ui_builder()).window_size((900.,600.)).title("窗口标题").menu(make_menu);
    AppLauncher::with_window(window)
        .log_to_console()
        .delegate(Delegate { windows: vec![] })
        .launch(AppData {
            data: String::new(),
        })
        .unwrap();
}

///
///构建主界面
///
fn ui_builder() -> impl Widget<AppData> {
    let data = lens!(AppData, data);
    let text = TextBox::new().lens(data).expand().controller(ContextMenuController {});
    text
}

/// 定义这个结构体，实现 Controller Trait，可以在实现对相关部件的事件处理
struct ContextMenuController;

///保存程序中所有的窗口的ID到Vec
struct Delegate {
    windows: Vec<WindowId>, 
}

///
///Controller 是一种管理子组件、重写或自定义其事件处理或更新行为的类型
///这里Hook主界面的的那个TexBox组件的事件，然后进行相应的处理
///
impl<W: Widget<AppData>> Controller<AppData, W> for ContextMenuController {
    fn event(
        &mut self,
        child: &mut W,
        ctx: &mut EventCtx,
        event: &Event,
        data: &mut AppData,
        env: &Env,
    ) {
        match event {//事件匹配
            Event::MouseDown(ref mouse) if mouse.button.is_right() => {//这个语法...比较丝滑, 但和其它语言确实不太一样
                ctx.show_context_menu(make_context_menu(), mouse.pos);
            }
            _ => child.event(ctx, event, data, env),
        }
    }
}

///这里是Hook顶层事件，可以Hook顶层事件进行自定义处理，可以取出cmd的值，判断是什么事件
impl AppDelegate<AppData> for Delegate {
    fn command(
        &mut self,
        ctx: &mut DelegateCtx,
        _target: Target,
        cmd: &Command,
        data: &mut AppData,
        _env: &Env,
    ) -> Handled {
        info!("u can handle system command here...");
        Handled::No
    }

    fn window_added(
        &mut self,
        id: WindowId,
        _handle: WindowHandle,
        _data: &mut AppData,
        _env: &Env,
        _ctx: &mut DelegateCtx,
    ) {
        info!("Window added, id: {:?}", id);
        self.windows.push(id);
    }

    fn window_removed(
        &mut self,
        id: WindowId,
        _data: &mut AppData,
        _env: &Env,
        _ctx: &mut DelegateCtx,
    ) {
        info!("Window removed, id: {:?}", id);
        if let Some(pos) = self.windows.iter().position(|x| *x == id) {
            self.windows.remove(pos);
        }
        if self.windows.len() == 0 {
            Application::global().quit(); // 这里如果所有窗口都已关闭，那就退出APP
        }
    }
}

///主菜单的构建
#[allow(unused_assignments)]
fn make_menu(_: Option<WindowId>, data: &AppData, _: &Env) -> Menu<AppData> {
    let mut base = Menu::empty();
    #[cfg(target_os = "macos")]
    {
        // base = druid::platform_menus::mac::menu_bar();
    }
    #[cfg(any(
    target_os = "windows",
    target_os = "freebsd",
    target_os = "linux",
    target_os = "openbsd"
    ))]
    {
        base = base.entry(druid::platform_menus::win::file::default());
    }
    base = base.entry(MenuItem::new("新建").on_activate(|_ctx, data: &mut AppData, _env| {
        info!("appdata:{}", data.data);
    }));
    base
}

///主界面中的TextBox组件的右键菜单的构建
fn make_context_menu() -> Menu<AppData> {
    Menu::empty()
        .entry(
            MenuItem::new("右键菜单1")
                .on_activate(|_ctx, data: &mut AppData, _env| {
                    info!("可以在这里添加右键菜单1的响应事件");
                }),
        )
}    
```
