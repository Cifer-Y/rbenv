# 用rbenv打理你的ruby环境

使用rbenv为你的应用选择一个ruby版本并保证开发环境和生产环境的一致。
rbenv和[Bundler](http://bundler.io/) 双剑合壁，让你不再苦恼ruby版本的升级和应用的部署。

**强势的开发表现** 只用在一个文件中设置一次ruby版本，保证不坑队友。
  应用太多？ruby版本太多？不用担心，你可以随时通过命令行和应用服务器比如 [Pow](http://pow.cx)
  重设ruby版本: 仅仅修改一个环境变量就可以了。

**稳定的生产环境表现** Your application's executables are its
  interface with ops. With rbenv and [Bundler
  binstubs](https://github.com/sstephenson/rbenv/wiki/Understanding-binstubs)
  you'll never again need to `cd` in a cron job or Chef recipe to
  ensure you've selected the right runtime. The Ruby version
  dependency lives in one place—your app—so upgrades and rollbacks are
  atomic, even when you switch versions.

**做好一件事** rbenv完全专注于切换ruby版本，简单可控。它有一个丰富的插件
  生态环境满足你一切需求。编译你自己的ruby版本或者使用[ruby-build][]插件
  自动构建。Specify per-application environment variables      
  with [rbenv-vars](https://github.com/sstephenson/rbenv-vars).
  查看[更多插件](https://github.com/sstephenson/rbenv/wiki/Plugins).

[**为什么选择rbenv而不是RVM?**](https://github.com/sstephenson/rbenv/wiki/Why-rbenv%3F)

## 目录

* [原理](#原理)
  * [理解PATH](#理解PATH)
  * [理解Shims](#understanding-shims)
  * [选择ruby版本](#理解Shims)
  * [定位ruby安装文件](#定位ruby安装文件)
* [安装](#安装)
  * [基本Github安装](#基本Github安装)
    * [升级](#upgrading)
  * [Homebrew安装](#Homebrew安装)
  * [rbenv怎样与你的shell勾搭](#rbenv怎样与你的shell勾搭)
  * [安装](#安装)
  * [卸载](#卸载)
* [命令参考](#命令参考)
  * [rbenv local](#rbenv-local)
  * [rbenv global](#rbenv-global)
  * [rbenv shell](#rbenv-shell)
  * [rbenv versions](#rbenv-versions)
  * [rbenv version](#rbenv-version)
  * [rbenv rehash](#rbenv-rehash)
  * [rbenv which](#rbenv-which)
  * [rbenv whence](#rbenv-whence)
* [环境变量](#环境变量)
* [开发](#开发)

## 原理

从宏观来看，rbenv通过将shim注入你的`PATH`来进行对ruby命令的拦截，根据
你的应用设置来决定ruby版本，并且把你的命令传递给正确版本的ruby。

### 理解PATH

当你敲一个命令的时候，比如`ruby`, ``rake`,你的操作系统会在一些目录里
去找叫这些名字的可执行文件.这些目录保存在一个名叫`PATH`的环境变量里,
不同目录之间用冒号分割:

    /usr/local/bin:/usr/bin:/bin

`PATH`中目录的搜索顺序是从左到右，所以排位靠前的目录中的命令会先于
排位靠后的目录中的命令执行。就上面的例子来说，搜索一个命令的顺序是
先`/usr/local/bin`再`/usr/bin`，最后 `/bin`。

### 理解Shims

rbenv通过把_shims_插入到你`PATH`的最前面来进行工作:

    ~/.rbenv/shims:/usr/local/bin:/usr/bin:/bin

通过一个叫做_rehashing_的过程，rbenv把shims维持在这个文件夹里去匹配
所有已安装的ruby版本的所有命令-`irb`, `gem`, `rake`, `rails`, `ruby`, 诸如此类。

Shims是轻量级的可执行文件，它仅仅是把你的命令传递给rbenv。所以，安装了rbenv之后，
当你跑一个命令，比如`rake`, 你的操作系统会做以下事情: 

* 搜索你的`PATH`去找一个叫做`rake`的可执行文件
* 在你的`PATH`最开始的地方找到一个叫做`rake`的rbenv shim
* 
* Run the shim named `rake`, which in turn passes the command along to
  rbenv

### Choosing the Ruby Version

When you execute a shim, rbenv determines which Ruby version to use by
reading it from the following sources, in this order:

1. The `RBENV_VERSION` environment variable, if specified. You can use
   the [`rbenv shell`](#rbenv-shell) command to set this environment
   variable in your current shell session.

2. The first `.ruby-version` file found by searching the directory of the
   script you are executing and each of its parent directories until reaching
   the root of your filesystem.

3. The first `.ruby-version` file found by searching the current working
   directory and each of its parent directories until reaching the root of your
   filesystem. You can modify the `.ruby-version` file in the current working
   directory with the [`rbenv local`](#rbenv-local) command.

4. The global `~/.rbenv/version` file. You can modify this file using
   the [`rbenv global`](#rbenv-global) command. If the global version
   file is not present, rbenv assumes you want to use the "system"
   Ruby—i.e. whatever version would be run if rbenv weren't in your
   path.

### Locating the Ruby Installation

Once rbenv has determined which version of Ruby your application has
specified, it passes the command along to the corresponding Ruby
installation.

Each Ruby version is installed into its own directory under
`~/.rbenv/versions`. For example, you might have these versions
installed:

* `~/.rbenv/versions/1.8.7-p371/`
* `~/.rbenv/versions/1.9.3-p327/`
* `~/.rbenv/versions/jruby-1.7.1/`

Version names to rbenv are simply the names of the directories in
`~/.rbenv/versions`.

## Installation

**Compatibility note**: rbenv is _incompatible_ with RVM. Please make
  sure to fully uninstall RVM and remove any references to it from
  your shell initialization files before installing rbenv.

If you're on Mac OS X, consider
[installing with Homebrew](#homebrew-on-mac-os-x).

### Basic GitHub Checkout

This will get you going with the latest version of rbenv and make it
easy to fork and contribute any changes back upstream.

1. Check out rbenv into `~/.rbenv`.

    ~~~ sh
    $ git clone https://github.com/sstephenson/rbenv.git ~/.rbenv
    ~~~

2. Add `~/.rbenv/bin` to your `$PATH` for access to the `rbenv`
   command-line utility.

    ~~~ sh
    $ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile
    ~~~

    **Ubuntu Desktop note**: Modify your `~/.bashrc` instead of `~/.bash_profile`.

    **Zsh note**: Modify your `~/.zshrc` file instead of `~/.bash_profile`.

3. Add `rbenv init` to your shell to enable shims and autocompletion.

    ~~~ sh
    $ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile
    ~~~

    _Same as in previous step, use `~/.bashrc` on Ubuntu, or `~/.zshrc` for Zsh._

4. Restart your shell so that PATH changes take effect. (Opening a new
   terminal tab will usually do it.) Now check if rbenv was set up:

    ~~~ sh
    $ type rbenv
    #=> "rbenv is a function"
    ~~~

5. _(Optional)_ Install [ruby-build][], which provides the
   `rbenv install` command that simplifies the process of
   [installing new Ruby versions](#installing-ruby-versions).

#### Upgrading

If you've installed rbenv manually using git, you can upgrade your
installation to the cutting-edge version at any time.

~~~ sh
$ cd ~/.rbenv
$ git pull
~~~

To use a specific release of rbenv, check out the corresponding tag:

~~~ sh
$ cd ~/.rbenv
$ git fetch
$ git checkout v0.3.0
~~~

If you've [installed via Homebrew](#homebrew-on-mac-os-x), then upgrade
via its `brew` command:

~~~ sh
$ brew update
$ brew upgrade rbenv ruby-build
~~~

### Homebrew on Mac OS X

As an alternative to installation via GitHub checkout, you can install
rbenv and [ruby-build][] using the [Homebrew](http://brew.sh) package
manager on Mac OS X:

~~~
$ brew update
$ brew install rbenv ruby-build
~~~

Afterwards you'll still need to add `eval "$(rbenv init -)"` to your
profile as stated in the caveats. You'll only ever have to do this
once.

### How rbenv hooks into your shell

Skip this section unless you must know what every line in your shell
profile is doing.

`rbenv init` is the only command that crosses the line of loading
extra commands into your shell. Coming from RVM, some of you might be
opposed to this idea. Here's what `rbenv init` actually does:

1. Sets up your shims path. This is the only requirement for rbenv to
   function properly. You can do this by hand by prepending
   `~/.rbenv/shims` to your `$PATH`.

2. Installs autocompletion. This is entirely optional but pretty
   useful. Sourcing `~/.rbenv/completions/rbenv.bash` will set that
   up. There is also a `~/.rbenv/completions/rbenv.zsh` for Zsh
   users.

3. Rehashes shims. From time to time you'll need to rebuild your
   shim files. Doing this automatically makes sure everything is up to
   date. You can always run `rbenv rehash` manually.

4. Installs the sh dispatcher. This bit is also optional, but allows
   rbenv and plugins to change variables in your current shell, making
   commands like `rbenv shell` possible. The sh dispatcher doesn't do
   anything crazy like override `cd` or hack your shell prompt, but if
   for some reason you need `rbenv` to be a real script rather than a
   shell function, you can safely skip it.

Run `rbenv init -` for yourself to see exactly what happens under the
hood.

### Installing Ruby Versions

The `rbenv install` command doesn't ship with rbenv out of the box, but
is provided by the [ruby-build][] project. If you installed it either
as part of GitHub checkout process outlined above or via Homebrew, you
should be able to:

~~~ sh
# list all available versions:
$ rbenv install -l

# install a Ruby version:
$ rbenv install 2.0.0-p247
~~~

Alternatively to the `install` command, you can download and compile
Ruby manually as a subdirectory of `~/.rbenv/versions/`. An entry in
that directory can also be a symlink to a Ruby version installed
elsewhere on the filesystem. rbenv doesn't care; it will simply treat
any entry in the `versions/` directory as a separate Ruby version.

### Uninstalling Ruby Versions

As time goes on, Ruby versions you install will accumulate in your
`~/.rbenv/versions` directory.

To remove old Ruby versions, simply `rm -rf` the directory of the
version you want to remove. You can find the directory of a particular
Ruby version with the `rbenv prefix` command, e.g. `rbenv prefix
1.8.7-p357`.

The [ruby-build][] plugin provides an `rbenv uninstall` command to
automate the removal process.

## Command Reference

Like `git`, the `rbenv` command delegates to subcommands based on its
first argument. The most common subcommands are:

### rbenv local

Sets a local application-specific Ruby version by writing the version
name to a `.ruby-version` file in the current directory. This version
overrides the global version, and can be overridden itself by setting
the `RBENV_VERSION` environment variable or with the `rbenv shell`
command.

    $ rbenv local 1.9.3-p327

When run without a version number, `rbenv local` reports the currently
configured local version. You can also unset the local version:

    $ rbenv local --unset

Previous versions of rbenv stored local version specifications in a
file named `.rbenv-version`. For backwards compatibility, rbenv will
read a local version specified in an `.rbenv-version` file, but a
`.ruby-version` file in the same directory will take precedence.

### rbenv global

Sets the global version of Ruby to be used in all shells by writing
the version name to the `~/.rbenv/version` file. This version can be
overridden by an application-specific `.ruby-version` file, or by
setting the `RBENV_VERSION` environment variable.

    $ rbenv global 1.8.7-p352

The special version name `system` tells rbenv to use the system Ruby
(detected by searching your `$PATH`).

When run without a version number, `rbenv global` reports the
currently configured global version.

### rbenv shell

Sets a shell-specific Ruby version by setting the `RBENV_VERSION`
environment variable in your shell. This version overrides
application-specific versions and the global version.

    $ rbenv shell jruby-1.7.1

When run without a version number, `rbenv shell` reports the current
value of `RBENV_VERSION`. You can also unset the shell version:

    $ rbenv shell --unset

Note that you'll need rbenv's shell integration enabled (step 3 of
the installation instructions) in order to use this command. If you
prefer not to use shell integration, you may simply set the
`RBENV_VERSION` variable yourself:

    $ export RBENV_VERSION=jruby-1.7.1

### rbenv versions

Lists all Ruby versions known to rbenv, and shows an asterisk next to
the currently active version.

    $ rbenv versions
      1.8.7-p352
      1.9.2-p290
    * 1.9.3-p327 (set by /Users/sam/.rbenv/version)
      jruby-1.7.1
      rbx-1.2.4
      ree-1.8.7-2011.03

### rbenv version

Displays the currently active Ruby version, along with information on
how it was set.

    $ rbenv version
    1.9.3-p327 (set by /Users/sam/.rbenv/version)

### rbenv rehash

Installs shims for all Ruby executables known to rbenv (i.e.,
`~/.rbenv/versions/*/bin/*`). Run this command after you install a new
version of Ruby, or install a gem that provides commands.

    $ rbenv rehash

### rbenv which

Displays the full path to the executable that rbenv will invoke when
you run the given command.

    $ rbenv which irb
    /Users/sam/.rbenv/versions/1.9.3-p327/bin/irb

### rbenv whence

Lists all Ruby versions with the given command installed.

    $ rbenv whence rackup
    1.9.3-p327
    jruby-1.7.1
    ree-1.8.7-2011.03

## Environment variables

You can affect how rbenv operates with the following settings:

name | default | description
-----|---------|------------
`RBENV_VERSION` | | Specifies the Ruby version to be used.<br>Also see [`rbenv shell`](#rbenv-shell)
`RBENV_ROOT` | `~/.rbenv` | Defines the directory under which Ruby versions and shims reside.<br>Also see `rbenv root`
`RBENV_DEBUG` | | Outputs debug information.<br>Also as: `rbenv --debug <subcommand>`
`RBENV_HOOK_PATH` | [_see wiki_][hooks] | Colon-separated list of paths searched for rbenv hooks.
`RBENV_DIR` | `$PWD` | Directory to start searching for `.ruby-version` files.

## Development

The rbenv source code is [hosted on
GitHub](https://github.com/sstephenson/rbenv). It's clean, modular,
and easy to understand, even if you're not a shell hacker.

Tests are executed using [Bats](https://github.com/sstephenson/bats):

    $ bats test
    $ bats test/<file>.bats

Please feel free to submit pull requests and file bugs on the [issue
tracker](https://github.com/sstephenson/rbenv/issues).


  [ruby-build]: https://github.com/sstephenson/ruby-build#readme
  [hooks]: https://github.com/sstephenson/rbenv/wiki/Authoring-plugins#rbenv-hooks
