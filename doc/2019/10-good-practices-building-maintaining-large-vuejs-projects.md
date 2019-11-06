# 构建Vue大型应用的10个最佳实践
   
   
   这些是我构建大型Vue项目得出的最佳实践，这些技巧将帮助你更高效的编码，并且使其更容易维护和协作。
    
    
在我今年的自由职业生涯中我有幸开发了一些大型Vue程序。我所说的这些项目使用了大量Vuex stores 😰 ，很多Vue组件（有数百个）和很多视图（pages）😄。对我而言这是非常有意义的经历，我发现了很多使扩展的方法。同时我还需要修复一些乱七八糟的错误用法。🍝

所以我将分享我的10个最佳实践，如果你有大型项目需要开发推荐你使用他们。

## 1. 使用Slots可以使你的组件更强大和便于理解。

最近我写了一篇文章[some important things you need to know regarding slots in Vue.js](https://www.telerik.com/blogs/all-you-need-to-know-about-slots-in-vuejs)。 主要讲了为什么使用Slots可以提高组件复用且易于维护。

🧐 但是这和大型Vue项目有啥关系呢。我将描绘一个使用他们解决你痛点的蓝图。

假如我要开发一个popup。起初看起来没有什么难点，仅仅是包括标题，描述和一些按钮。 所以把所有东西都当作props不就完了吗。 我用了三个自定义props，并且click按钮的时候触发一个事件。 就是这么简单！ 😅

随着项目的不断发展，业务需要显示许多其他新内容：表单字段，不同的按钮（取决于显示在哪个页面上）、卡片、页脚和列表。通过添加更多的props可以解决这个问题。😩但是随着业务发展，组件变的太复杂了。因为他包含了很多子组件，需要触发很多个事件。🌋我遇到了坑爹的问题，修改了一个功能后影响了其他page上的功能。我创造了一个怪物而不是一个可维护的组件！🤖

但是，一开始使用slots可能会更好。最终我使小组件来重构这个组件。使他更容易维护、好理解、好扩展。

```html

<template>
  <div class="c-base-popup">
    <div v-if="$slot.header" class="c-base-popup__header">
      <slot name="header">
    </div>
    <div v-if="$slot.subheader" class="c-base-popup__subheader">
      <slot name="subheader">
    </div>
    <div class="c-base-popup__body">
      <h1>{{ title }}</h1>
      <p v-if="description">{{ description }}</p>
    </div>
    <div v-if="$slot.actions" class="c-base-popup__actions">
      <slot name="actions">
    </div>
    <div v-if="$slot.footer" class="c-base-popup__footer">
      <slot name="footer">
    </div>
  </div>
</template>

<script>
export default {
  props: {
    description: {
      type: String,
      default: null
    },
    title: {
      type: String,
      required: true
    }
  }
}
</script>

```
根据经验在我看来，由熟练使用slots的开发人员构建项目对将来项目的可维护性有更重要的意义。

> ⚠️ 经验规则，当子组件中使用了父组件的props时，应该使用slots。 

## 2.好好组织的你Vuex store

通常，一个新的Vue开发人员学习Vuex是因为两个问题：

* 当一个组件需要重一个比较远的组件（嵌套层级或者兄弟组件：译者注）访问数据时。
* 需要持有一个销毁组件的数据时。

这样他创建第一个Vuex store，学习moudles的用法并且组织程序时。💡
问题是创建modules时没有单一模式可循。我强烈建议一定要想清楚如何组织他们。据我所知，很多人更喜欢按照功能来组织他们（我也是：译者注）。例如：

*   Auth.
*   Blog.
*   Inbox.
*   Settings.

就我来说，用从API获得的数据模型组织更容易理解。例子：

*   Users
*   Teams
*   Messages
*   Widgets
*   Articles

用那个取决于自己，但是从长远来看组织良好的Vuex store更具生产力。这样也容易是新人融入并且继承你的初衷。

## 3.用Actions调用api和提交数据。

我的大部分（不是全部）API调用都在Vuex的actions中，你一定想知道为什么这样做。🤨

🤷🏼‍简单的说是因为拉取数据时需要在store中commit。还有就是他提供了我喜欢的封装和复用。其他原因就是：

* 如果我在两个地方（blog和home page）获取第一个页面的数据。我只需使用不同的参数调用即可得到数据，初次之外没有重复的代码被调用。
* 如果需要创建一些逻辑来避免重复拉取数据，只需要在一个地方拉取一次。除了给服务器减负以外，我还可以在任何地方使用这些数据。
* 我可以在actions中最终Mixpanel事件，基于维护性使问题变的容易分析。我的代码中大部分action中只有一个Mixpanel调用，😂我不必关注数据和发送这种工作方式太爽了。


## 4.使用mapState、mapGetters、mapMutations和mapAction精简代码。

当你在组件中访问state/getters或者调用actions/mutations时通常需要创建多个计算属性。使用mapState，mapGetters，mapMutations和mapActions可以将来自store modules中的数据集中在一个地方，这样可以精简代码并且更好理解。

```javascript
// NPM
import { mapState, mapGetters, mapActions, mapMutations } from "vuex";

export default {
  computed: {
    // Accessing root properties
    ...mapState("my_module", ["property"]),
    // Accessing getters
    ...mapGetters("my_module", ["property"]),
    // Accessing non-root properties
    ...mapState("my_module", {
      property: state => state.object.nested.property
    })
  },

  methods: {
    // Accessing actions
    ...mapActions("my_module", ["myAction"]),
    // Accessing mutations
    ...mapMutations("my_module", ["myMutation"])
  }
};

```

你想要的这里都有[Vuex官方文档](https://vuex.vuejs.org/guide/modules.html)。🤩

##5. 快使用API Factories

我通常会创建this.$api助手，可以在任何地方访问我的API入口。我的项目根目录有一个`api`文件夹有我的所有类（如下）。

```
api
├── auth.js
├── notifications.js
└── teams.js
```
每一个都是一类接口的分组，这是我在Nuxt应用中使用插件的方式初始化。（和标准Vue应用程序中的过程非常相似）。

```
// PROJECT: API
import Auth from "@/api/auth";
import Teams from "@/api/teams";
import Notifications from "@/api/notifications";

export default (context, inject) => {
  if (process.client) {
    const token = localStorage.getItem("token");
    // Set token when defined
    if (token) {
      context.$axios.setToken(token, "Bearer");
    }
  }
  // Initialize API repositories
  const repositories = {
    auth: Auth(context.$axios),
    teams: Teams(context.$axios),
    notifications: Notifications(context.$axios)
  };
  inject("api", repositories);
};

```

```
export default $axios => ({
  forgotPassword(email) {
    return $axios.$post("/auth/password/forgot", { email });
  },

  login(email, password) {
    return $axios.$post("/auth/login", { email, password });
  },

  logout() {
    return $axios.$get("/auth/logout");
  },

  register(payload) {
    return $axios.$post("/auth/register", payload);
  }
});

```

这样我可以方便的在组件或Vuex操作中调用他们，如下：

```
export default {
  methods: {
    onSubmit() {
      try {
        this.$api.auth.login(this.email, this.password);
      } catch (error) {
        console.error(error);
      }
    }
  }
};

```

## 6. 使用$config访问环境变量(模板中特别有用)。

项目中定义了一些全局配置变量：

```
config
├── development.json
└── production.json
```

我通常使用`this.$config`获取，尤其是当我在模板中时。 一如既往扩展Vue对象非常容易：

```
// NPM
import Vue from "vue";

// PROJECT: COMMONS
import development from "@/config/development.json";
import production from "@/config/production.json";

if (process.env.NODE_ENV === "production") {
  Vue.prototype.$config = Object.freeze(production);
} else {
  Vue.prototype.$config = Object.freeze(development);
}

```

## 7.遵守一个commit命名规则。

在项目发展的过程中，经常需要关注组件的变更历史。如果团队中有人没有遵守commit惯例，那么将很难理解他们所做的事情。

我总是使用并推荐[Angular commit message guidelines](https://github.com/angular/angular/blob/master/CONTRIBUTING.md#-commit-message-guidelines)。我所有的项目中都会使用他，通常团队中的其他人也会发现他的好处。

遵守这些规则使commit更加可读，在查看项目历史时使得commit更加容易追踪。简言之，他是这样用的：

```
git commit -am "<type>(<scope>): <subject>"

# Here are some samples
git commit -am "docs(changelog): update changelog to beta.5"
git commit -am "fix(release): need to depend on latest rxjs and zone.js"

```

看他们的[README file](https://github.com/angular/angular/blob/master/CONTRIBUTING.md#-commit-message-guidelines)更新更多。

## 8.始终在生产环境中冻结Package的版本。

我知道...所有软件包都应遵循[ the semantic versioning rules.](https://semver.org/)。但实际情况并非如此。😅

为了避免一个依赖影响了整个项目在半夜被拖起来，冻结所有程序包的版本可以使你一觉睡到天明并且工作愉快。 😇

这很简单：避免以^开头的版本：

```
{
  "name": "my project",

  "version": "1.0.0",

  "private": true,

  "dependencies": {
    "axios": "0.19.0",
    "imagemin-mozjpeg": "8.0.0",
    "imagemin-pngquant": "8.0.0",
    "imagemin-svgo": "7.0.0",
    "nuxt": "2.8.1",
  },

  "devDependencies": {
    "autoprefixer": "9.6.1",
    "babel-eslint": "10.0.2",
    "eslint": "6.1.0",
    "eslint-friendly-formatter": "4.0.1",
    "eslint-loader": "2.2.1",
    "eslint-plugin-vue": "5.2.3"
  }
}

```

## 9. 显示一个大的数据时应该是用Vue虚拟滚动条。

在页面中显示多行或需要循环大量数据时，你已经注意到该页面渲染速度很快变慢。 要解决此问题，您可以使用[vue-virtual-scoller](https://github.com/Akryum/vue-virtual-scroller)。

```
npm install vue-virtual-scroller
```

他只渲染列表中的可见项并且复用组件和dom元素，以使其尽可能高效。 如此简单就像一个魔法！ ✨

```
<template>
  <RecycleScroller
    class="scroller"
    :items="list"
    :item-size="32"
    key-field="id"
    v-slot="{ item }"
  >
    <div class="user">
      {{ item.name }}
    </div>
  </RecycleScroller>
</template>
```

## 10.追踪第三方程序包的大小

多人合作一个项目时，如果没人关注安装的依赖包数量很快变的难以置信。为了避免程序变慢（尤其是在移动网络环境），我这VSC中使用[import cost package](https://marketplace.visualstudio.com/items?itemName=wix.vscode-import-cost)这样就可以编辑器中看到导入的包有多大，并且找出大的原因。

例如在最近的项目中，导入了整个lodash库（压缩后24kB）。 有啥子问题？ 仅仅使用cloneDeep方法。 通过import cost package找到了问题，我们通过以下方式解决了该问题：

```
npm remove lodash
npm install lodash.clonedeep
```
在使用的地方导入：

```
import cloneDeep from "lodash.clonedeep";
```

> ⚠️ 为了进一步优化，我们还可以使用Webpack Bundle Analyzer包通过树状图来可视化Webpack输出文件的大小。


在大型Vue项目中还有其他最佳实践吗？ 请在下面的评论中告诉我，或者在Twitter @RifkiNada上与我联系。 🤠

---
*   我的blog:[neverland.github.io](https://neverland.github.io/)
*   我的email[enix@foxmail.com](https://neverland.github.io/doc/2019/enix@foxmail.com)
