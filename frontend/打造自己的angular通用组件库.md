## 前言 ##
在项目开发过程中，经常会抽取出一些通用性的组件，在angular项目中通常放于shared-module目录下，这样在该项目就可以引入shared-module并使用。进一步地，如果我们希望shared-module可以跨项目使用，就需要打造一个自己的shared-module-library。
本文将利用Angular CLI和[ng-packagr](https://www.npmjs.com/package/ng-packagr)一步步构建出一个shared-module-library。

## 正文 ##
1. 新建一个ng项目
```
ng new ng-packagr-test
```

npm install完毕后，启动ng serve，可以看到现在就是一个普通的ng项目。

2. 创建共享的module和component

以创建一个header.module和header.component为例。
可以用cli命令或者手动创建
```
ng generate module modules/header

ng generate component modules/header
```

现在的目录结构大概如下图：
![image](https://github.com/dyh333/blog/blob/master/assets/imgs/1-2-1.png)

打开header.component.html并替换为：
```
<h1>
  <ng-content></ng-content>
</h1>
```

__导出组件__：
```
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { HeaderComponent } from './header.component';
@NgModule({
  imports: [
    CommonModule
  ],
  declarations: [
    HeaderComponent
  ],
  exports: [
    HeaderComponent // <-- this!
  ]
})
export class HeaderModule { }
```

exports确保将该模块中导出的组件可以被其它模块引入并使用。

添加HeaderModule到app.module中：
```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
// import our module 
import { HeaderModule } from './modules/header/header.module';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    HeaderModule // import it into our @NgModule block
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

此时，就可以在app.component.html使用HeaderComponent组件了：
```
<app-header>Such Header</app-header>
```

再次启动ng serve，可以看到HeaderComponent已经显示在页面上了
![image](https://github.com/dyh333/blog/blob/master/assets/imgs/1-2-2.png)

自此，我们已经创建了一个待共享的header模块及相应的组件。

__ng-packagr__
ng-packagr可以将ng项目编译并打包成一个umd规范的library，以便可以被其它的ng项目所使用。

安装ng-packagr：
```
npm install ng-packagr --save-dev 
```
在项目根目录下添加两个文件 __ng-package.json__ 和 __public_api.ts__。

ng-package.json内容：
```
{
  "$schema": "./node_modules/ng-packagr/ng-package.schema.json",
  "lib": {
    "entryFile": "public_api.ts"
  }
}
```

在public_api.ts中导出header.module.ts：
```
export * from './src/app/modules/header/header.module'
```

在package.json文件中添加packagr脚本命令，并将private属性设置为false：
```
"scripts": {
  "ng": "ng",
  "start": "ng serve",
  "build": "ng build",
  "test": "ng test",
  "lint": "ng lint",
  "e2e": "ng e2e",
  "packagr": "ng-packagr -p ng-package.json"
},
"private": false
```

运行packagr脚本命令
```
npm run packagr
```

结束后会生成一个dist文件夹，就是我们需要的library包了。还可以进一步将dist打包成tgz文件：
```
cd dist
npm pack
```
dist文件夹里就会多出一个ng-packagr-test-0.0.0.tgz，名称和版本号均取自package.json。

自此，我们就可以通过磁盘相对路径来安装自己的library了，如：
```
npm install ../some-relative-path/dist/ng-packagr-test-0.0.0.tgz
```

还可以通过自己的npm账号发布到npmjs，前提是确保包名是唯一的：
```
npm publish dist
```

这是我发布的[ngx-packagr-test](https://www.npmjs.com/package/ngx-packagr-test)。

发布后也可以通过npmjs来下载安装了：
```
npm install ngx-packagr-test
```

上述测试项目已共享到[github](https://github.com/dyh333/ng-packagr-test)，欢迎下载。

## 后记 ##
angular6官方已增强了library的构建。我自己也试了一下，发现ng generate library命令必须是在一个ng项目中执行才行。所以构建出来的一个library目录结构如下：
![image](https://github.com/dyh333/blog/blob/master/assets/imgs/1-2-3.png)

等于是将library和src/app放在同一项目中了，目前还不知道如何将library单独打包出来。

## 参考 ##
[Building an Angular 4 Component Library with the Angular CLI and ng-packagr](https://medium.com/@nikolasleblanc/building-an-angular-4-component-library-with-the-angular-cli-and-ng-packagr-53b2ade0701e)