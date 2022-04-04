---
title: remix框架初体验
tags: remix,react
date: 2022-04-04 21:49:42
---


其实很早就对[remix][remix]感兴趣了，不过那时候的remix是封闭的，只给付费人员使用，不舍花钱的我只能馋着了。不过去年10月份[remix][remix]开源了，也有机会一探究竟了

[remix][remix]作者是[react-router]的作者[Ryan Florence](https://twitter.com/ryanflorence)和[MICHAEL JACKSON](https://twitter.com/mjackson)。最近又加入了React社区非常有名的[Kent](https://twitter.com/kentcdodds)。Kent的[个人博客](https://kentcdodds.com/blog?q=remix)上也有蛮多文章介绍[remix][remix]的，感兴趣的可以去看一下。我会从实际项目出发，讲一讲[remix][remix]的设计理念和特性

## 项目介绍

先简单介绍一下这个项目。最近一直在学英语，学习方法是找一些youtube的视频跟读，就需要一边播放视频，一边听写出文本，然后跟着读。同时我也希望有个地方能够纪录一下我的练习，所以就做了这么一个网站[https://shadowing-henna.vercel.app/](https://shadowing-henna.vercel.app/)

项目比较简单，就是一个列表页，一个详情页/编辑页，以及一个新增页。但能覆盖基本的CRU - create, read, update场景。我们跟着这些页面，来看看[remix][remix]的特性

### 列表页

```jsx
// app/routes/index.jsx
import { useLoaderData, Link } from 'remix';
import supabase from '~/utils/supabase';
import { format } from '~/utils/date';

export const loader = async () => {
  // supabase是我们的数据库
  const { data, error } = await supabase
    .from('shadows')
    .select('id,title,created_at')
    .order('created_at', { ascending: false })

  if (error) throw error.message;
  return data
}

export default function Index() {
  const execises = useLoaderData();

  return (
    <div className="root">
      {
        execises.map(item => <Item key={item.id} {...item} />)
      }
    </div>
  );
}

function Item({ id, title, created_at }) {
  return (
    <article className="item">
      <Link className="item-title" to={`/${id}`}>{title}</Link>
      <p>{format(created_at)}</p>
    </article>
  )
}
```

以上就是列表页的代码了。可以看到比较特别是，除了`export`出react组件外，还`export`出了一个`loader`方法，这就是[remix][remix]获取服务端数据的方法。

remix是一个全栈的react框架，`loader`是直接运行在服务端的，也就是说我们可以在`loader`里使用服务端能力，比如读取本地文件，访问数据库等等。像我们这个地方就是直接读取数据库里的数据然后返回。组件里的`useLoaderData`方法就是用来获取`loader`中返回的数据的。

当我们的组件依赖服务端数据，需要做异步请求时，一般都会需要考虑几个问题 - loading怎么处理？error怎么处理？服务端数据子组件需要用的话怎么处理？

#### loading

remix走的是SSR这条道，而且即使在客户端页面跳转时，remix也会保证在`loader`执行成功后，才会开始渲染我们的组件。Goodbye `if (loading) return <div>loaindg...</div>`

#### error

当接口报错时又如何处理呢？[remix][remix]拥抱了react的做法 - [ErrorBoundary](https://reactjs.org/docs/error-boundaries.html)。我们可以`export`出一个`ErrorBoundary`组件，处理异常

```jsx
// app/routes/index.jsx
...

export function ErrorBoundary({ error }) {
  return (
    <div className="error-container">
      出错了 - {error.message}
    </div>
  );
}
```

#### 数据共享

该路由下的所有组件，都可以直接用`useLoaderData`获取这份数据。基本上我们也不再需要引入其他的状态管理方案了

### 新增页

[remix][remix]又是如何做数据更新的呢？大家还记得大明湖畔的`<form action="">`吗？


```jsx
import { redirect, useTransition, useActionData, json } from 'remix';
import { useState, useEffect } from 'react';
import supabase from '~/utils/supabase';

export const action = async ({ request }) => {
  const body = await request.formData();
  const title = body.get('title');
  const content = body.get('content')
  const vid = body.get('vid')

  const { data, error } = await supabase.from('shadows')
    .insert([
      { title, content, vid, type: 1 },
    ])
  
  if (error)  return json(error.message || `Something went wrong!`, {
    status: 500
  })

  return redirect(`/${data[0].id}`);
}

export default function Index() {
  const [vid, setVid] = useState();

  const handleVidChange = e => {
    const url = new URL(e.target.value);
    setVid(url.searchParams.get('v'))
  }

  return (
    <form method="post" className="form">
      <textarea name="content" className="editor" />
      <div>
        <input required name="title" type="text" placeholder="title" />
        <input autoFocus required name="url" placeholder="youtube video link" type="text" onChange={handleVidChange} />
        <input name="vid" hidden value={vid} />
        <button className="button" type="submit">Save</button>
        <div className="video">
          {vid && <lite-youtube videoid={vid} />}
        </div>
      </div>
    </form>
  );
}
```

普通的`form`，唯一不同的是`export`出了一个`action`方法，`action`也是运行在服务端的。这就是[remix][remix]处理表单的方法，拥抱标准。当表单提交时，服务端会收到客户端请求，`action`方法被执行，通过`request.formData`可以拿到表单的数据。这里的`request`, `reponse`等用的都是[`fetch API`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)的标准实现

这一点也体现出[remix][remix]的一个设计理念，拥抱web标准 - Work with, not against, the foundations of the web: Browsers, HTTP, and HTML. It’s always been good and it's gotten really good in the last few years.

拥抱标准有什么好处呢？这段代码在js被禁用是依然可以正常工作 - `vid`这部分。也就意味着在js加载的过程中，用户就可以和我们的应用交互了

拥抱标准的另一个好处就是，你学习的知识是可迁移的，而不是跟某个框架绑定的。举个简单例子，大家现在可能会用各种表单库或ui库去做表单开发，你学习的知识这个库的api。当你没法使用这个库时，这部分知识就作用不大了。举个具体例子，当一个开发中后台的同学需要去开发h5，且涉及表单的时候？

这也是[remix][remix]文档上所写的 - 在用[remix][remix]开发应用时，可能大部分时候你都在看MDN的文档


## remix pattern

```jsx
// 读数据
export async function loader({ request }) {
  ...
}

// 表单提交 - 写数据
export async function action({ request }) {
  ...
}

export default function () {
  // 取数据  
  const data = useLoaderData();

  ...
}
```

以上就基本是`remix`给我们规范的读数据和写数据的方法，同时上文也提到过，这些只能在`route module`中使用。配合react router提供的nested routes，避免了一个页面所有的数据都在集中在一个`loader`中的情形

这个设计我认为有几点非常好

1. 作为一个全栈的react框架，既拥有的服务端的能力，同时又和前端的代码够内聚，使用和实现维护在一个地方，开发体验非常好
2. 数据和ui做了很好地分离 - 组件变得非常简单，基本就和一个接收props的组件一样，且代码都是同步的，不用在异步和同步建切换
3. 不用在组件内处理loading和error，用更声明式的方式替代过程式的 `if (loading)`，`if(errro)`代码
4. 拥抱标准

当然，我上面说的都是DX开发体验层面的，[remix][remix]还提供了很多用户体验层面的优化，这个在后续的文章中会为大家介绍，不在这里展开


### 如何引入样式

目前用下来，[remix][remix]中处理样式的方法，是我唯一感到开发体验不太友好的地方

```jsx
import styles from "~/styles/index.css";

export function links() {
  return [{ rel: "stylesheet", href: styles }];
}
```

如上代码所示，你可以通过在路由组件中`export`出`links`方法来引入样式。对于路由组件的样式来讲，这样写是可以接受的。但是对于通用组件的样式呢？比如如下这个例子

```css
/* app/components/button/style.css */

[data-button] {
  border: solid 1px;
  background: white;
  color: #454545;
}
```

```jsx
// app/components/button/index.jsx
import styles from "./styles.css";

export const links = () => [
  { rel: "stylesheet", href: styles },
];

export const Button = React.forwardRef(
  ({ children, ...props }, ref) => {
    return <button {...props} ref={ref} data-button />;
  }
);
```

如果你想使用这个组件，你需要在路由组件中显示的引入这个样式

```jsx
// app/routes/index.jsx
import styles from "~/styles/index.css";
import {
  Button,
  links as buttonLinks,
} from "~/components/button";

export function links() {
  return [
    ...buttonLinks(),
    { rel: "stylesheet", href: styles },
  ];
}
```

另外[remix][remix]默认是不支持css预处理器和后处理器的，如果想引入的话，需要自己处理成`css`引入。
以及按[remix][remix]的设计，`import`出来的`style`需要是一个url。所以和目前社区上的一些`css in js`库可能无法兼容。

由于目前我还没有在样式复杂的项目上体验[remix][remix]，所以具体的开发体验还有待验证

### 浏览器兼容性

国内当前的环境，当开发C端，特别是移动端页面时，还是的考虑兼容性问题。[remix][remix]只能运行在支持[ES Modules](https://caniuse.com/es6-module)的浏览器上。对于国内的环境是一个挑战

不过如上文中提到的，及时禁用js，[remix](remix)运用依然是可以运行，提供基本的能力的。这也是[remix][remix]遵循的另一个理念，[Progressive enhancement - 渐进增强](https://en.wikipedia.org/wiki/Progressive_enhancement)，即每个人都可以访问到内容和使用基本的功能，而对于使用设备更好的浏览器的用户，提供更好的交互体验

不过这个能不能说服PM同学，就得靠大家了~~~

好啦，这就是最近一段时间使用[remix][remix]的体验，整体来讲是非常好一个提个框架，理念上，设计上，实现上都值得大家关注。后续会为大家带来更多原理分析和实战分享


[remix]: https://remix.run/
[react-router]: https://reactrouter.com/