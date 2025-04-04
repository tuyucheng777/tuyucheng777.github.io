---
layout: post
title:  包含前端应用程序的Spring Security OAuth-授权代码流程
category: spring-security
copyright: spring-security
excerpt: Spring Security
---

## 1. 概述

在本教程中，我们将继续我们的[Spring Security OAuth系列](https://www.baeldung.com/spring-security-oauth)，为授权码流构建一个简单的前端。

请记住，这里的重点是客户端；阅读[Spring REST API + OAuth2 + AngularJS](https://www.baeldung.com/rest-api-spring-oauth2-angular-legacy)文章-查看授权和资源服务器的详细配置。

## 2. 授权服务器

在进入前端之前，我们需要在授权服务器配置中添加客户端详细信息：

```java
@Configuration
@EnableAuthorizationServer
public class OAuth2AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {

    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
               .withClient("fooClientId")
               .secret(passwordEncoder().encode("secret"))
               .authorizedGrantTypes("authorization_code")
               .scopes("foo", "read", "write")
               .redirectUris("http://localhost:8089/")
    // ...
```

请注意我们如何启用授权码授予类型，其简单详细信息如下：

- 客户端ID是fooClientId
- 作用域是foo、read和write
- 重定向URI是http://localhost:8089/(我们将为前端应用程序使用端口8089)

## 3. 前端

现在，让我们开始构建简单的前端应用程序。

由于我们将在这里为我们的应用程序使用Angular 6，因此我们需要在Spring Boot应用程序中使用frontend-maven-plugin插件：

```xml
<plugin>
    <groupId>com.github.eirslett</groupId>
    <artifactId>frontend-maven-plugin</artifactId>
    <version>1.6</version>

    <configuration>
        <nodeVersion>v8.11.3</nodeVersion>
        <npmVersion>6.1.0</npmVersion>
        <workingDirectory>src/main/resources</workingDirectory>
    </configuration>

    <executions>
        <execution>
            <id>install node and npm</id>
            <goals>
                <goal>install-node-and-npm</goal>
            </goals>
        </execution>
        <execution>
            <id>npm install</id>
            <goals>
                <goal>npm</goal>
            </goals>
        </execution>
        <execution>
            <id>npm run build</id>
            <goals>
                <goal>npm</goal>
            </goals>
            <configuration>
                <arguments>run build</arguments>
            </configuration>
        </execution>
    </executions>
</plugin>
```

当然，我们需要先在我们的机器上安装[Node.js](https://nodejs.org/en/)；我们将使用Angular CLI为我们的应用程序生成基础：

```shell
ng new authCode
```

## 4. Angular模块

现在，让我们详细讨论一下Angular模块。

这是我们的简单AppModule：

```javascript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { HttpClientModule } from '@angular/common/http';
import { RouterModule }   from '@angular/router';
import { AppComponent } from './app.component';
import { HomeComponent } from './home.component';
import { FooComponent } from './foo.component';

@NgModule({
    declarations: [
        AppComponent,
        HomeComponent,
        FooComponent
    ],
    imports: [
        BrowserModule,
        HttpClientModule,
        RouterModule.forRoot([
            { path: '', component: HomeComponent, pathMatch: 'full' }], {onSameUrlNavigation: 'reload'})
    ],
    providers: [],
    bootstrap: [AppComponent]
})
export class AppModule { }
```

我们的模块由三个组件和一个服务组成，我们将在以下部分讨论它们。

### 4.1 应用组件

让我们从根组件AppComponent开始：

```typescript
import {Component} from '@angular/core';

@Component({
    selector: 'app-root',
    template: `<nav class="navbar navbar-default">
    <div class="container-fluid">
        <div class="navbar-header">
            <a class="navbar-brand" href="/">Spring Security Oauth - Authorization Code</a>
        </div>
    </div>
</nav>
<router-outlet></router-outlet>`
})

export class AppComponent {}
```

### 4.2 主页组件

接下来是我们的主页组件HomeComponent：

```typescript
import {Component} from '@angular/core';
import {AppService} from './app.service'

@Component({
    selector: 'home-header',
    providers: [AppService],
    template: `<div class="container" >
    <button *ngIf="!isLoggedIn" class="btn btn-primary" (click)="login()" type="submit">Login</button>
    <div *ngIf="isLoggedIn" class="content">
        <span>Welcome !!</span>
        <a class="btn btn-default pull-right"(click)="logout()" href="#">Logout</a>
        <br/>
        <foo-details></foo-details>
    </div>
</div>`
})

export class HomeComponent {
    public isLoggedIn = false;

    constructor(
        private _service:AppService){}

    ngOnInit(){
        this.isLoggedIn = this._service.checkCredentials();
        let i = window.location.href.indexOf('code');
        if(!this.isLoggedIn && i != -1){
            this._service.retrieveToken(window.location.href.substring(i + 5));
        }
    }

    login() {
        window.location.href = 'http://localhost:8081/spring-security-oauth-server/oauth/authorize?response_type=code&client_id=' + this._service.clientId + '&redirect_uri='+ this._service.redirectUri;
    }

    logout() {
        this._service.logout();
    }
}
```

注意：

- 如果用户未登录，则只会出现登录按钮
- 登录按钮将用户重定向到授权URL
- 当用户使用授权码重定向回来时，我们将使用此代码检索访问令牌

### 4.3 Foo组件

我们的第3个也是最后一个组件是FooComponent；它显示从资源服务器获取的Foo资源：

```typescript
import { Component } from '@angular/core';
import {AppService, Foo} from './app.service'

@Component({
    selector: 'foo-details',
    providers: [AppService],
    template: `<div class="container">
    <h1 class="col-sm-12">Foo Details</h1>
    <div class="col-sm-12">
        <label class="col-sm-3">ID</label> <span>{{foo.id}}</span>
    </div>
    <div class="col-sm-12">
        <label class="col-sm-3">Name</label> <span>{{foo.name}}</span>
    </div>
    <div class="col-sm-12">
        <button class="btn btn-primary" (click)="getFoo()" type="submit">New Foo</button>        
    </div>
</div>`
})

export class FooComponent {
    public foo = new Foo(1,'sample foo');
    private foosUrl = 'http://localhost:8082/spring-security-oauth-resource/foos/';

    constructor(private _service:AppService) {}

    getFoo(){
        this._service.getResource(this.foosUrl+this.foo.id)
            .subscribe(
                data => this.foo = data,
                error =>  this.foo.name = 'Error');
    }
}
```

### 4.4 应用服务

现在，让我们看一下AppService：

```typescript
import {Injectable} from '@angular/core';
import { Cookie } from 'ng2-cookies';
import { HttpClient, HttpHeaders } from '@angular/common/http';
import { Observable } from 'rxjs/Observable';
import 'rxjs/add/operator/catch';
import 'rxjs/add/operator/map';

export class Foo {
    constructor(
        public id: number,
        public name: string) { }
}

@Injectable()
export class AppService {
    public clientId = 'fooClientId';
    public redirectUri = 'http://localhost:8089/';

    constructor(
        private _http: HttpClient){}

    retrieveToken(code){
        let params = new URLSearchParams();
        params.append('grant_type','authorization_code');
        params.append('client_id', this.clientId);
        params.append('redirect_uri', this.redirectUri);
        params.append('code',code);

        let headers = new HttpHeaders({'Content-type': 'application/x-www-form-urlencoded; charset=utf-8', 'Authorization': 'Basic '+btoa(this.clientId+":secret")});
        this._http.post('http://localhost:8081/spring-security-oauth-server/oauth/token', params.toString(), { headers: headers })
            .subscribe(
                data => this.saveToken(data),
                err => alert('Invalid Credentials')
            );
    }

    saveToken(token){
        var expireDate = new Date().getTime() + (1000 * token.expires_in);
        Cookie.set("access_token", token.access_token, expireDate);
        console.log('Obtained Access token');
        window.location.href = 'http://localhost:8089';
    }

    getResource(resourceUrl) : Observable<any>{
        var headers = new HttpHeaders({'Content-type': 'application/x-www-form-urlencoded; charset=utf-8', 'Authorization': 'Bearer '+Cookie.get('access_token')});
        return this._http.get(resourceUrl,{ headers: headers })
            .catch((error:any) => Observable.throw(error.json().error || 'Server error'));
    }

    checkCredentials(){
        return Cookie.check('access_token');
    }

    logout() {
        Cookie.delete('access_token');
        window.location.reload();
    }
}
```

让我们在这里快速回顾一下我们的实现情况：

- checkCredentials()：检查用户是否已登录
- retrieveToken()：使用授权码获取访问令牌
- saveToken()：将访问令牌保存在Cookie中
- getResource()：使用其ID获取Foo详细信息
- logout()：删除访问令牌Cookie

## 5.运行应用程序

为了运行我们的应用程序并确保一切正常运行，我们需要：

- 首先，在端口8081上运行授权服务器
- 然后，在端口8082上运行资源服务器
- 最后，运行前端

我们需要先构建应用程序：

```shell
mvn clean install
```

然后将目录更改为src/main/resources：

```shell
cd src/main/resources
```

然后在端口8089上运行我们的应用程序：

```shell
npm start
```

## 6. 总结

我们学习了如何使用Spring和Angular 6为授权码流构建一个简单的前端客户端。