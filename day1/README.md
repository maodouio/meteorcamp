# 第一周周一

## Meteor基础(1)

- 日期：20160118
- 地点：线上(Moxtra)
- 讲师：李明 & 田思源

----


### Meteor 7 原则


Meteor.js 不同于其他框架的地方。

- 数据在线上（Data on the Wire）

"Don't send HTML over the network. Send data and let the client decide how to render it."

Insensible code that include HTML on JSON output is not a fault of any framework. In fact you can just add handlebars on your Ruby on Rails application and that should sort things out. It's used as a default in Meteor.js.

- 单一语言（One Language: JavaScript）

Initially daunting for someone who uses Ruby on Rails and who believes in JavaScript minimalism, but it makes a lot of sense just as writing an app entirely on Ruby or Python makes sense.

- 数据库无处不在（Database Everywhere）

"Use the same transparent API to access your database from the client or the server." 

This just means there's one way of getting data from any table. This is how you define a list of items on either the server or client:

```
Lists = new Meteor.Collection("lists");
```

- 延迟补偿（Latency Compensation）

"On the client, use prefetching and model simulation to make it look like you have a zero-latency connection to the database."

Prefetching is faster than it is in Rails naturally.

- 全栈响应性（Full Stack Reactivity）

"Make realtime the default. All layers, from database to template, should make an event-driven interface available."

This is the behaviour needed by some people I talked to. They don't like to keep on clicking on buttons. The user should just type and see results quickly.

- 拥抱生态环境（Embrace the Ecosystem）

"Meteor is open source and integrates, rather than replaces, existing open source tools and frameworks."

You can use Node.js libraries for most of the things that Meteor.js lacks.

- 简单即效率（Simplicity Equals Productivity）

"The best way to make something seem simple is to have it actually be simple. Accomplish this through clean, classically beautiful APIs."

### 基本概念

- Isomorphic JavaScript

只写一套代码，但可以跑在服务器端和客户端。

[Isomorphic JavaScript](http://isomorphic.net/javascript) apps are JavaScript applications that can run both client-side and server-side. The backend and frontend share the same code. 

- Template

helpers, events 都是 template 里的东西。

helpers 是很多键值对，用来提供数据。值不一定是固定的，可以是一个 function 返回的值。然后 html 里就可以访问这些数据了。如果 helper 是一个带参数的 function. 那么 html 里也可以传参给这个 function。

events 显而易见是用来注册 event handler 的。这里的好处是，events 是定义在某个 template 下的，即是说，如果两个 template 里的 class 名一样，也不会发生事件同时被触发的问题，因为 events 本身被限定了作用域（这种事情一定是我这种取名困难的人才会担心）。event handler 接受两个参数，第一个是 event 本身，第二个是 template instance. template instance 即是包含触发事件元素的 template. 里面有一些有用的方法，比如 find, findAll 来找 dom 。这些方法都跟 jQuery 很像，所以比较容易理解和使用。

而 template 本身还有一些方法，比如 onCreated, onRendered, onDestroyed, 用来在 template 创建，渲染，销毁时做一些操作。

- Session

Session 里可以存储任意键值对，它们在客户端部分操作，可以理解为客户端部分的数据库，而且不会对服务器造成影响。利用 Session 也可以让 View 自动更新，这里用来实现这个功能的就是 Tracker。

- Tracker

tracker 的用途是当 Session 里的变量，数据查询或其他数据源发生改变是，自动返回新的 template 或其他函数。这个过程是自动的，看起来有点像 Angular 里的双向绑定，不需要手动控制。我对这里还是有点疑问的，就像 Angular , 如果数据越来越多，需要 track 的数据也会越来越多，那么会不会影响效率呢，比如渲染变慢。

如果需要更细节的控制，Tracker.autorun() 是用来干这个的，它接受一个回调函数，当它的依赖数据改变后，这个函数就会被再次运行。

- Collections

Collection 是用来存储数据的，它模拟了 MongoDB 的 collection. 在创建 Collection 时，如果给了名字的参数，那么这个 Collection 就是 persistent 的，会存储到服务器，并推送给客户端。不然它只是一个 local collection. 不会同步到服务器。Collection 支持的几种方法都跟 MongoDB 支持的方法基本一样，update, insert, remove, find 之类的。

- Methods

Methods 是定义在服务器端的一些方法，可以在客户端进行调用，使用它的原因是要对数据做一些复杂的操作，比如数据验证。这里因为调用的是服务器端的方法，所以有网络请求，会造成一些延时，这时 Meteor 会做出补偿，即是在客户端就预测服务器端会返回的结果，直接在客户端做出 UI 上的改变，等服务器端返回值后，如果没有不同，则不会变动，如果不同，则再改变 UI。

- Publish and Subscribe

通常情况下，可能并不希望服务器端的所有数据都暴露给客户端，因此就需要 Pub/Sub 来解决这个问题，服务器端决定 Pub 什么东西，然后在客户端进行 Sub。

- Environment

Meteor 有不同的环境，服务器端和客户端，它提供了 isClient, isServer 来判断环境，让不同环境的代码写在不同的地方。这样就很清晰地分离了前后端，然后结合 Collection 在不同环境的统一 API, 让数据的操作和同步变得容易管理。startUp 方法则是让应用在启动时可以执行一些操作。



### Meteor 程序结构

- 目录布局

```
lib/                      # common code like collections and utilities
lib/methods.js            # Meteor.methods definitions
lib/constants.js          # constants used in the rest of the code

client/compatibility      # legacy libraries that expect to be global
client/lib/               # code for the client to be loaded first
client/lib/helpers.js     # useful helpers for your client code
client/body.html          # content that goes in the <body> of your HTML
client/head.html          # content for <head> of your HTML: <meta> tags, etc
client/style.css          # some CSS code
client/<feature>.html     # HTML templates related to a certain feature
client/<feature>.js       # JavaScript code related to a certain feature

server/lib/permissions.js # sensitive permissions code used by your server
server/publications.js    # Meteor.publish definitions

public/favicon.ico        # app icon

settings.json             # configuration data to be passed to meteor --settings
mobile-config.js          # define icons and metadata for Android/iOS

```

- 例子

```
tsy@localhost:~$ meteor create --example todos
Created a new Meteor app in 'todos'.          

To run your new app:                          
  cd todos                                    
  meteor                                      
                                              
If you are new to Meteor, try some of the learning resources here:
  https://www.meteor.com/learn                
                                              
tsy@localhost:~$ cd todos
tsy@localhost:~/todos$ tree .
.
├── client
│   ├── head.html
│   ├── lib
│   │   └── jquery.touchwipe.js
│   ├── stylesheets
│   │   ├── globals
│   │   │   ├── base.import.less
│   │   │   ├── button.import.less
│   │   │   ├── form.import.less
│   │   │   ├── icon.import.less
│   │   │   ├── layout.import.less
│   │   │   ├── link.import.less
│   │   │   ├── list-items.import.less
│   │   │   ├── menu.import.less
│   │   │   ├── message.import.less
│   │   │   ├── nav.import.less
│   │   │   └── notification.import.less
│   │   ├── main.less
│   │   └── util
│   │       ├── fontface.import.less
│   │       ├── helpers.import.less
│   │       ├── lesshat.import.less
│   │       ├── reset.import.less
│   │       ├── text.import.less
│   │       ├── typography.import.less
│   │       └── variables.import.less
│   └── templates
│       ├── app-body.html
│       ├── app-body.js
│       ├── app-not-found.html
│       ├── app-not-found.import.less
│       ├── auth.import.less
│       ├── auth-join.html
│       ├── auth-join.js
│       ├── auth-signin.html
│       ├── auth-signin.js
│       ├── lists-show.html
│       ├── lists-show.import.less
│       ├── lists-show.js
│       ├── loading.html
│       ├── loading.import.less
│       ├── todos-item.html
│       └── todos-item.js
├── lib
│   ├── collections.js
│   └── router.js
├── mobile-config.js
├── public
│   ├── apple-touch-icon-precomposed.png
│   ├── favicon.png
│   ├── font
│   │   ├── OpenSans-Light-webfont.eot
│   │   ├── OpenSans-Light-webfont.svg
│   │   ├── OpenSans-Light-webfont.ttf
│   │   ├── OpenSans-Light-webfont.woff
│   │   ├── OpenSans-Regular-webfont.eot
│   │   ├── OpenSans-Regular-webfont.svg
│   │   ├── OpenSans-Regular-webfont.ttf
│   │   └── OpenSans-Regular-webfont.woff
│   ├── icon
│   │   ├── todos.eot
│   │   ├── todos.svg
│   │   ├── todos.ttf
│   │   └── todos.woff
│   └── img
│       └── logo-todos.svg
├── resources
│   ├── icons
│   │   ├── icon-29x29@2x.png
│   │   ├── icon-29x29.png
│   │   ├── icon-36x36.png
│   │   ├── icon-40x40@2x.png
│   │   ├── icon-40x40.png
│   │   ├── icon-48x48.png
│   │   ├── icon-50x50@2x.png
│   │   ├── icon-50x50.png
│   │   ├── icon-57x57@2x.png
│   │   ├── icon-57x57.png
│   │   ├── icon-60x60@2x.png
│   │   ├── icon-60x60.png
│   │   ├── icon-72x72@2x.png
│   │   ├── icon-72x72.png
│   │   ├── icon-76x76@2x.png
│   │   ├── icon-76x76.png
│   │   └── icon-96x96.png
│   └── splash
│       ├── splash-1024x768@2x.png
│       ├── splash-1024x768.png
│       ├── splash-1280x720.png
│       ├── splash-200x320.png
│       ├── splash-320x200.png
│       ├── splash-320x480@2x.png
│       ├── splash-320x480.png
│       ├── splash-320x568@2x.png
│       ├── splash-480x320.png
│       ├── splash-480x800.png
│       ├── splash-720x1280.png
│       ├── splash-768x1024@2x.png
│       ├── splash-768x1024.png
│       └── splash-800x480.png
└── server
    ├── bootstrap.js
    └── publish.js

15 directories, 88 files
tsy@localhost:~/todos$
```

### 实操