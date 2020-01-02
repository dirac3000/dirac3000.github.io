---
id: 164
title: Rust Language Environment
date: 2017-09-11T16:00:59+02:00
author: Alvaro
layout: post
guid: http://alvarom.com/?p=164
permalink: /2017/09/11/rust-language-environment/
ci_cpt_video_url:
  - ""
ci_cpt_show_gallery:
  - ""
image: /http://alvarom.com/wp-content/uploads/2017/09/rust-logo-512x512-blk-200x200.png
categories:
  - Programming
  - Technology
---
Rust is a rather young language getting more and more popular as a &#8220;safe, concurrent, practical language&#8221;. I wanted to learn how to code in rust, and I wanted to setup a development environment to make the learning path easier.

Here are the notes on how I set up my Rust + Visual Studio Code environment. <!--more-->I wrote them while setting them up on a Ubuntu Linux 16.04. It might work on other platforms too. I tried to do it on Windows, but it is slower and it ends up being a pain, especially when it comes down to debugging (rust-lldb is not available on the msvc toolchain) [EDIT: apparently 

[someone](http://www.shadercat.com/setting-up-a-rust-development-environment-on-windows-10/) had more chance than me with this environment].

  * Download and run the rust installer called rustup: _curl https://sh.rustup.rs -sSf | sh_
  * I chose to install the nightly toolchain, because it has already happened to that some packages work only on the nightly toolchain and they produce errors on the stable one (sigh).
  * Once everything is installed and you have the PATH correctly set up, you should be able to run for example the rust compiler (rustc) and the package manager (cargo) from your shell (installed on ~/.cargo/bin). To check if everything went fine, you can try to make a hello world project: 
    <pre>cargo new hello_world --bin
cd hello_world
cargo run</pre>

  * In order to debug your binaries, you might want to install LLDB. To do that just type sudo _apt install lldb_. Another tool you might need is python&#8217;s six, you can install via _pip install six_. Once these steps are done you should be able to debug your binaries (try doing _rust-lldb target/debug/hello_world_)
  * Whatever your choice of IDE/editor will be, there are few tools that might help you when working on Rust code. Some of the tools this plug-ins might benefit from are racer, rustsym, rustfmt and RLS: 
      * **Racer** is a code completion helper. To install it, you can use cargo: _cargo install racer_ (it might take few minutes to download and compile all dependencies).
      * **Rustsym** is a tool to query symbols from rust code. To install it: _cargo install rustsym_
      * **Rustfmt** is a code formatter. To install: _cargo install rustfmt_
      * **RLS** is a language server. That means Rust compiler can run in the background, while the IDE might ask it information about the code you are writing. To install it, you can follow the information on [their website](https://github.com/rust-lang-nursery/rls), that sums up to this: 
        <pre>rustup component add rls --toolchain nightly</pre>
        
        <pre>rustup component add rust-analysis --toolchain nightly</pre>
        
        <pre>rustup component add rust-src --toolchain nightly</pre>

I found that **Visual Studio Code**, together with few plug-ins, make a nice IDE. There might be alternatives (IntelliJ IDEA, Eclipse or Vim) but I was happy with VSCode. To install it, download it from [their site](https://code.visualstudio.com). VSCode by itself will not be enough until you install some handy plugins:

  * _VSCode-Rust_ to have the great rust support and turn VSCode into a good Rust IDE.
  * _LLDB_ Debugger provides a nice interface with the debugger.

I had a silly message telling me that it could not detect the default toolchain, you just close it will propose you one of the installed toolchains.  
Once all these tools are installed, you should have a nice IDE to work with Rust. To be honest, for some trivial changes I still go back to vim and the shell sometimes, but having an IDE makes life simpler most of the time, especially when you are a newbie at it.